Graylog *Docker* container
==================================

Detailed documentation can be found [here](http://docs.graylog.org/en/latest/pages/installation/docker.html).

## Multi-node setup 
The above documentation will show you all of the options for how to run Graylog in a single container and single
node setup.  

If you want to run this on a multi-node configuration the following instructions will show you how to do that.

This will show you how to run a multi-node configuration in three ways
* On 1 server with Docker server installed
* On 3 servers with Docker server installed
* On a 3 node CoreOS cluster

### Running on 1 Docker Servers
You can mimic a multi-node configuration with Docker even though you have 1 server.  We do this by running
multiple container exposing ports all on one server.  From the perspective of the application it looks like
 each process is on another server or at least it is reaching out to something else beside localhost.
 
#### Starting 3 Elasticsearch containers

Setup the host IP variable.  This is the IP of the server.

    export HOST_IP=<HOST IP>
    
Create data output directories.  The data from Elasticsearch will be saved here for persistent.

    mkdir -p /opt/elasticsearch/node1/data
    mkdir -p /opt/elasticsearch/node2/data
    mkdir -p /opt/elasticsearch/node3/data
    chmod -R 777 /opt/elasticsearch
    
Start the 3 Elasticsearch containers:
    
    docker run -d \
    -p 9200:9200 \
    -p 9300:9300 \
    -e CLUSTER=graylog2 \
    -e UNICAST_HOSTS=${HOST_IP}:9300,${HOST_IP}:9301,${HOST_IP}:9302 \
    -e NODE_NAME=elasticsearch1 \
    -v /opt/elasticsearch/node1/data:/data \
    itzg/elasticsearch
    
    docker run -d \
    -p 9201:9200 \
    -p 9301:9300 \
    -e CLUSTER=graylog2 \
    -e UNICAST_HOSTS=${HOST_IP}:9300,${HOST_IP}:9301,${HOST_IP}:9302 \
    -e NODE_NAME=elasticsearch2 \
    -v /opt/elasticsearch/node2/data:/data \
    itzg/elasticsearch
    
    docker run -d \
    -p 9202:9200 \
    -p 9302:9300 \
    -e CLUSTER=graylog2 \
    -e UNICAST_HOSTS=${HOST_IP}:9300,${HOST_IP}:9301,${HOST_IP}:9302 \
    -e NODE_NAME=elasticsearch3 \
    -v /opt/elasticsearch/node3/data:/data \
    itzg/elasticsearch
    
We have exposed a different port for each Elasticsearch container and also passed it into the container
so that each containers knows where the other hosts are and at what port they are using.  This allows us
to run all 3 Elasticsearch nodes on one machine.

There is probably no reason you want to do this except for testing or development purposes.

Once they are started, you can check the health of the nodes by going to this URL:

    http://${HOST_IP}:9200/_cluster/health?pretty

#### Starting Graylog Server

    export 

    docker run \
    --name graylog-server \
    -t \
    -p 12900:12900 \
    -p 12201:12201 \
    -p 12201:12201/udp \
    -p 4001:4001 \
    -e GRAYLOG_SERVER=true \
    -e GRAYLOG_NODE_ID=de305d54-75b4-431b-adb2-eb6b9e83743 \
    -e GRAYLOG_SERVER_SECRET=somesecretsaltstring \
    -e GRAYLOG_TIMEZONE=America/Los_Angeles \
    -e GRAYLOG_PASSWORD=SeCuRePwD \
    -e GRAYLOG_USERNAME=admin \
    -v /opt/graylog/data:/var/opt/graylog/data \
    -v /opt/graylog/logs:/var/log/graylog \
    -v /opt/graylog/plugins:/opt/graylog/plugin \
    graylog2/allinone
    
#### Starting Graylog web interface

    docker run \
    --name graylog-web \
    -t \
    -p 9000:9000 \
    -p 443:443 \
    -e GRAYLOG_MASTER=${HOST_IP} \
    -e GRAYLOG_WEB=true \
    graylog2/allinone

### Running on 3 servers with Docker server installed

TBD

### Running on a 3 node CoreOS cluster

Start a unit that will set the IPs of the nodes that will be running Elasticsearch into etcd

    fleetctl start set-elasticsearch-nodes@{1..3}.service
    
Start the Elasticsearch units

    fleetctl start elasticsearch@{1..3}.service
    









