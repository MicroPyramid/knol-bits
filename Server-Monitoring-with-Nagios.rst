We use Nagios to monitor our servers

and this is our guide to install it in ubuntu with Icinga and pnp4nagios graphs


In the Server:

Update the server

1) apt-get update and apt-get upgrade

install the mysql package:

2) apt-get install mysql-server && mysql-client

install the nagios package:

3) apt-get install nagios3 && apt-get install nagios-nrpe-plugin

 [OPTIONAL] to change the password for the nagiosadmin user enter:

4) sudo htpasswd /etc/nagios3/htpasswd.users nagiosadmin
 
Now in the browser enter http://ipaddress/nagios3

Nagios is good to go now install ICINGA with pnp4nagios(graphs):

6) aptitude install icinga pnp4nagios

If asked for enable external commands continue with 'yes'.

Configuring Icinga to send email:

 Install postfix:

7) apt-get install postfix

8) nano /etc/icinga/objects/contacts_icinga.cfg

change root@localhost to your-email-address

9) nano /etc/icinga/icinga.cfg    and change the following variable

            process_performance_data=1

            broker_module=/usr/lib/pnp4nagios/npcdmod.o config_file=/etc/pnp4nagios/npcd.cfg

10) nano /etc/default/npcd and set

            RUN="yes"

11) to show graphs for hosts, nano /etc/icinga/objects/generic-host_icinga.cfg and add

           action_url  /pnp4nagios/graph?host=$HOSTNAME$

12) to show graphs for services, nano /etc/icinga/objects/generic-service_icinga.cfg and add:

           action_url  /pnp4nagios/graph?host=$HOSTNAME$&srv=$SERVICEDESC$

 13) nano /etc/apache2/conf.d/pnp4nagios.conf  and change the “nagios3” directory to “icinga”:


           AuthUserFile /etc/icinga/htpasswd.users
Restarts:
             service apache2 restart       
             
             service npcd start                         
             
             service icinga restart                     

Now do the following changes in the remote host/Client:

1)install nrpe server which constantly sends information to nrpe plugin in the icinga server
     
           sudo apt-get install nagios-nrpe-server

2) nano /etc/nagios/nrpe.cfg

           allowed_hosts=localhost, nagios server ip address

           command[check_all_disks]=/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -e
           
           command[check_memory]=/usr/lib/nagios/plugins/check_memory.sh -w 85 -c 90

 3) nano /usr/lib/nagios/plugins/
           
            touch check_memory.sh
 
            chmod +x check_memory.sh
 
            nano check_memory.sh     and copy the following 

ps: also repeat the above third step in the server. 

 #!/bin/bash

# Nagios plugin to check memory consumption
# Excludes Swap and Caches so only the real memory consumption is considered
. /usr/lib/nagios/plugins/utils.sh
# set default values for the thresholds
WARN=90
CRIT=95
while getopts "c:w:h" ARG; do
 case $ARG in
  w) WARN=$OPTARG;;
  c) CRIT=$OPTARG;;
  h) echo "Usage: $0 -w <warning threshold> -c <critical threshold>"; exit;;
 esac
done
MEM_TOTAL=`free | fgrep "Mem:" | awk '{print $2}'`;
MEM_USED=`free | fgrep "/+ buffers/cache" | awk '{print $3}'`;
PERCENTAGE=$(($MEM_USED*100/$MEM_TOTAL))
echo "${PERCENTAGE}% ($((($MEM_USED)/1024)) of $((MEM_TOTAL/1024)) MB) used";
if [ $PERCENTAGE -gt $CRIT ]; then
 exit $STATE_CRITICAL;
elif [ $PERCENTAGE -gt $WARN ]; then
 exit $STATE_WARNING;
else
 exit $STATE_OK;
fi
           
           

Back to server:

to add hosts copy contents of localhost_icinga.cfg and paste it in new file

          cp /etc/icinga/objects/localhost_icinga.cfg /etc/icinga/objects/exampleremotehost.cfg 

and edit the contents in the exampleremothost.cfg

Add host details in it. for example my server is abc.xyz.com

define host{
        use                     generic-host            ; Name of host template to use
        host_name         abc.xyz.com
        alias                   abc.xyz.com 
        address              ip address of abc.xyz.com
        }

Add services to the hosts edit the above file

# Define a service to check the number of currently logged in

define service{
        use                             generic-service         ; Name of service template to use
        host_name                  abc.xyz.com
        service_description      Current Users
        check_command          check_nrpe!check_users!20!50
        }

# Define a service to check the number of currently running procs

define service{
        use                             generic-service         ; Name of service template to use
        host_name                  abc.xyz.com
        service_description     Total Processes
        check_command         check_nrpe!check_total_procs!250!400
        }

# Define a service to check the load on the remote machine.

define service{
        use                             generic-service         ; Name of service template to use
        host_name                  abc.xyz.com
        service_description      Current Load
        check_command          check_nrpe!check_load!5.0!4.0!3.0!10.0!6.0!4.0
        }

#Check Memory usage/Ram usage on the remote machine

 define service {
        use                             generic-service
        host_name                  erp.micropyramid.com
        service_description     Memory Usage
        check_command         check_nrpe_1arg!check_memory
         }
ps: unfortunately we were unable to integrate graph to the check memory usage service. But we'll soon figure out a way to make it. 

# Define a service to check the disk space of the root partition

define service{
        use                             generic-service         ; Name of service template to use
        host_name                  abc.xyz.com
        service_description     Disk Space
        check_command         check_nrpe!check_all_disks!ip address of remote machine
        }

 now restart icinga, apache2 and enjoy

 1)service icinga restart

 2)service apache2 restart

TroubleShoots:

If you get an error when trying to execute external commands.

error: Could not stat() command file ‘/var/lib/icinga/rw/icinga.cmd’!

Just follow these:

1)service icinga stop

2)dpkg-statoverride --update --add nagios www-data 2710 /var/lib/icinga/rw/

3)dpkg-statoverride --update --add nagios nagios 751 /var/lib/icinga/

4)service icinga start

If you get error regarding file not found just create the file in the specified directory and give permissions
 to create file 

   1)touch filename

 to change permissions

  2) chmod +x filename

Flap Detection:

Icinga has flap detection which suppresses email notifications when a host or service frequently changes its state.Flap detection is experimental and not yet stable so i suggest to disable flap detection
edit nagios.cfg file and change enable_flap_detection to 0

nano /etc/icinga/icinga.cfg 

enable_flap_detection=0
