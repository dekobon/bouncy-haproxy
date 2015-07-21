Bouncy HAProxy
==============

This Docker images aims to overcome some of the shortcomings of the official
HAProxy image. Understandably, the official image needs to conform to general
standards that may be at odds with an operationalized image. This image differs
in the following ways:

### Differences Between library/haproxy

 1. It is using phusion-base image in order to run as a multi-process Docker
    container.
 2. It is running a syslog daemon and cron in order to properly handle log
    output from HAProxy because HAProxy doesn't support a production-ready
    stand-alone mode that outputs logs to STDOUT.
 3. It provides a default volume so that your configuration can be persisted
    even if you remove this container.
 4. It supports restarting HAProxy without shutting down the container. Since
    HAProxy needs to have the old PIDs shutdown in order reload the 
    configuration this often results in HAProxy exiting entirely and thereby
    ending the container management process (PID 1). This makes bouncing the
    load balancer configuration very difficult.
 5. The multi-process model on this container will allow those who want to
    extend it to be able to write their own processes that monitor an endpoint
    or a DOCKER_HOST and then modify the configuration. Next, they can then
    bounce HAProxy and have a new server pool loaded.

### Setup Hints

 1. You probably want to mount a data volume on /etc/haproxy/config in order to make your 
    HAProxy configuration persistance across reboots.
 2. You will need to modify the default configuration file /etc/haproxy/config/haproxy.cfg
    to conform to your particular setup.
 3. If you add servers inside of the ### start/end pool ### comment block you can use 
    utilities to template in linked servers into your configuration.
 4. Take a look at update-lb-pool.sh for an example of a script that you could
    run from a machine configured with a DOCKER_HOST variable. This script will
    allow you to update your haproxy.cfg to include all hosts that match a given
    prefix (as is predictably created by docker-compose). For example, you could add
    all instances of your site container by doing: 
    ./update_lb_pool.sh cluster_site 80 cluster_haproxy_1

### Runtime Hints

 1. You can reload the proxy configuration at runtime by running the script: /usr/local/bin/bounce_haproxy
 
