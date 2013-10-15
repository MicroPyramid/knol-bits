Celery
======

Celery is a task queue with focus on real-time processing,while also supports task scheduling.
Task queues are used  as  mechanisms to distribute work across multiple threads or machines.
A task queues input is a unit of work called a task,dedicated worker processes and constantly moniter the queue for new work to perform.
celery communicates via messages using a  broker to mediate between workers and clients.To initiate a task client puts a message on the queue,then the broker delivers that message to a worker.

Usage
=====
- Choosing and installing a message broker.
- Installing Celery and creating your first task
- Starting the worker and calling tasks.
- Keeping track of tasks as they transition through different states, and inspecting.

Choosing a Broker
-----------------
Celery requires a solution to send and receive messages, usually this comes in the form of a separate service called a message broker.

There are several choices available, including:

- RabbitMQ

    RabbitMQ is feature-complete, stable, durable and easy to install. It’s an excellent choice for a production environment. Detailed information about using RabbitMQ with Celery:

    http://docs.celeryproject.org/en/latest/getting-started/brokers/rabbitmq.html#broker-rabbitmq

- Redis

    Redis is also feature-complete, but is more susceptible to data loss in the event of abrupt termination or power failures.

- Using a database

    Using a database as a message queue is not recommended, but can be sufficient for very small installations. Your options include:

    Using SQLAlchemy

    Using the Django Database

- Other brokers

    In addition to the above, there are other experimental transport implementations to choose from, including Amazon SQS, Using MongoDB and IronMQ.

Installing Celery
-----------------
Celery is on the Python Package Index (PyPI), so it can be installed with standard Python tools like pip or easy_install:

.. code-block:: bash

    $ pip install celery

    
Application
-------------
The first thing you need is a 'tasks.py' and create a task using ``@task`` decorator, it must be possible for other modules to import it.


Let’s create the file tasks.py:

.. code-block:: python

    from celery.decorators import task
    @task
    def add(x, y):
        return x + y

You defined a single task, called add, which returns the sum of two numbers.

Starting the worker
-------------------
You now you can run the worker by executing the following command :

$ python manage.py celery worker --loglevel=info


Calling the task
----------------
To call our task you can use the ``delay()`` method which gives greater control of the task execution (see Calling Tasks):

First import the corresponding task from 'tasks.py'

.. code-block:: python
    from tasks import add

call the ``delay()`` method with the task

.. code-block:: python
    add.delay(4, 4)
    
The task has now been will be processed by the worker you are going to start, and you can verify that by looking at the workers console output.

Calling a task returns an AsyncResult instance, which can be used to check the state of the task, wait for the task to finish or get its return value (or if the task failed, the exception and traceback). But this isn’t enabled by default, and you have to configure Celery to use a result backend.

Keeping Results
---------------
If you want to keep track of the tasks’ states, Celery needs to store or send the states somewhere. There are several built-in result backends to choose from: SQLAlchemy/Django ORM, Memcached, Redis, AMQP (RabbitMQ), and MongoDB – or you can define your own.

For this example you will use the Mongo DB result backend, which sends states as messages. The backend is specified via CELERY_RESULT_BACKEND setting :

CELERY_RESULT_BACKEND = "mongodb"
CELERY_MONGODB_BACKEND_SETTINGS = {
    "host": "192.168.1.100",
    "port": 30000,
    "database": "mydb",
    "taskmeta_collection": "my_taskmeta_collection",
}


To read more about result backends please see http://docs.celeryproject.org/en/latest/userguide/tasks.html#task-result-backends.

Now with the result backend configured, let’s call the task again. This time you’ll hold on to the AsyncResult instance returned when you call a task:

result = add.delay(4, 4)
The ready() method returns whether the task has finished processing or not:

>>> result.ready()
False
You can wait for the result to complete, but this is rarely used since it turns the asynchronous call into a synchronous one:

>>> result.get(timeout=1)
8
In case the task raised an exception, get() will re-raise the exception, but you can override this by specifying the propagate argument:

>>> result.get(propagate=True)
If the task raised an exception you can also gain access to the original traceback:

>>> result.traceback
...
See celery.result for the complete result object reference.

Configuration
Celery, like a consumer appliance doesn’t need much to be operated. It has an input and an output, where you must connect the input to a broker and maybe the output to a result backend if so wanted. But if you look closely at the back there’s a lid revealing loads of sliders, dials and buttons: this is the configuration.

The default configuration should be good enough for most uses, but there’s many things to tweak so Celery works just the way you want it to. Reading about the options available is a good idea to get familiar with what can be configured. You can read about the options in the the Configuration and defaults reference.

The configuration can be set on the app directly or by using a dedicated configuration module. As an example you can configure the default serializer used for serializing task payloads by changing the CELERY_TASK_SERIALIZER setting:

celery.conf.CELERY_TASK_SERIALIZER = 'json'

For larger projects using a dedicated configuration module is useful, in fact you are discouraged from hard coding periodic task intervals and task routing options, as it is much bett

CELERY_ROUTES = {
    'tasks.add': 'low-priority',
}
Or instead of routing it you could rate limit the task instead, so that only 10 tasks of this type can be processed in a minute (10/m):


CELERY_ANNOTATIONS = {
    'tasks.add': {'rate_limit': '10/m'}
}
If you are using RabbitMQ, Redis or MongoDB as the broker then you can also direct the workers to set a new rate limit for the task at runtime:

$ celery control rate_limit tasks.add 10/m
worker.example.com: OK
    new rate limit set successfully
See Routing Tasks to read more about task routing, and the CELERY_ANNOTATIONS setting for more about annotations, or Monitoring and Management Guide for more about remote control commands, and how to monitor what your workers are doing.

Running the worker with supervisor
----------------------------------
In production you will want to run the worker in the background as a daemon. To do this you need to use the tools provided like supervisord.

First, you need to install supervisor in your virtualenv and generate a configuration file.

.. code-block:: python

    $ pip install supervisor
    $ cd /path/to/your/project
    $ echo_supervisord_conf > supervisord.conf

Next, just add the following section in configuration file:

.. code-block:: bash
    [program:celeryd]
    command=python manage.py celery worker -l info 
    stdout_logfile=/path/to/your/logs/celeryd.log
    stderr_logfile=/path/to/your/logs/celeryd.log
    autostart=true
    autorestart=true
    startsecs=10
    stopwaitsecs=600

It's a simplified version of the Celery supervisor configuration file, adapted to work with virtualenvs.

Usage

Just run supervisord in your project directory.

.. code-block:: bash

    $ supervisord

Running supervisor during startup or booting time
-------------------------------------------------
	
create a file /etc/init.d/supervisord and configure your actual supervisord.conf in which celery is configured in DAEMON_ARGS as follows

.. code-block:: bash
    DAEMON_ARGS="-c /path/to/supervisord.conf"

to run it

.. code-block:: bash
    sudo chmod +x /etc/init.d/supervisord

and to automatically schedule it, do

.. code-block:: bash
    sudo update-rc.d supervisord defaults

To Stop and Start the service

.. code-block:: bash
    service supervisord stop
    service supervisord start

using upstart
-------------
Create a new file /etc/init/supervisor.conf. Its content should look like this:

.. code-block:: bash

    description "supervisor"
    start on runlevel [2345]
    stop on runlevel [!2345]
    respawn
    chdir /path/to/supervisord
    exec supervisord

Note that we’re using the same supervisord configuration file we used before. No changes there…

We can now start and stop supervisord with the following commands

.. code-block:: bash

    $ sudo stop supervisor 
    $ sudo start supervisor 
