Celery
======

Celery is a task queue with focus on real-time processing,while also supports task scheduling.
Task queues are used  as  mechanisms to distribute work across multiple threads or machines.
A task queues input is a unit of work called a task,dedicated worker processes and constantly moniter the queue for new work to perform.
celery communicates via messages using a  broker to mediate between workers and clients.To initiate a task client puts a message on the queue,then the broker delivers that message to a worker.
Usage
=====
