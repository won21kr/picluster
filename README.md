# PiCluster

PiCluster is a simple way to manage Docker containers on multiple hosts. I created this
because I found Docker Swarm not that good and Kubernetes was too difficult to install currently on ARM.

## Features

* Run commands in parallel across Nodes
* Heartbeat for services
* Easily build and orchestrate Docker images across nodes

## Prerequisites

* Docker
* Node.js


## Server Installation

##### 1. Modify config.json with your desired layout.

This is the core config file. Under layout, each row contains an IP
address of the node to run the container on, the name for the container, and the Docker arguments.

The heartbeat section of config.json lists the node, container name, and the port to monitor. If the port can not be connected to, PiCluster will restart the failed image.

The token section lets you define a random string that will be used for authentication with the agents. If the token in config.json
does not match the token in the agent's config (auth.json), no commands will be run on the agent.

The docker section defines where your Dockerfile's are. The format for the Docker folder should be like this:
dockerfiles/imagename/Dockerfile

```
{
  "token":"1234567890ABCDEFGHJKLMNOP",
  "docker": "/root/docker",
  "layout": [
    {"node":"192.168.0.100", "mysql":"-p 3306:3306 mysql","nginx":"nginx"},
    {"node":"192.168.0.102", "openvpn":"-p 1194:1194 openvpn"}
  ],
  "hb": [
    {"node":"192.168.0.100","mysql":"3306", "nginx": "80"},
    {"node":"192.168.0.102","openvpn":"1194"}
  ]
}

```

##### 2. Running the Application


The following environment variables need to be set:
* PORT - Port that the server listens on
* AGENTPORT - Port that the agent listens on

```
export PORT='9000'
export AGENTPORT='9001'
cd server
npm install
node server.js
```

## Agent Installation

The server will send commands to be executed on the agents nodes. The agent should be installed on each host in the cluster. You can
run the server and agent on the same node since they are listening on different ports.

##### 1. Modify auth.json with the token from the server.

```
{
  "token":"1234567890ABCDEFGHJKLMNOP"
}
```

##### 2. Running the Application

The following variable needs to be set:
* AGENTPORT - Port that the agent listens on
```
export AGENTPORT='9001'
cd agent
npm install
node agent.js
```

## Configuring and using the client "pictl"

Pictl is a bash client to easily control the cluster. It will make all the HTTP requests using curl.

#### 1. The following variables need to be set in the file:

* server - IP address of the server
* port - PORT that the server uses

#### 2. Using the client

To get a list of accepted arguments:
```
pictl
```

To build all of the Docker images from your config:
```
pictl build
```

To run all of the Docker images from your config:
```
pictl run
```

To stop all of the Docker images from your config:
```
pictl stop
```

To execute a command on all of the hosts
```
pictl exec "command"
```

Display all of the Docker images on each host
```
pictl images
```

To view the current log
```
pictl log
```

## Using Systemd for Server and Agent Processes

The systemd folder containers the service files and scripts to make PiCluster start at boot time.

#### 1. Modify the .service files in the systemd folder

For each .service file, change ExecStart and ExecStop to refelct the location of the PiCluster folder.
```
ExecStart=/bin/bash /root/picluster/systemd/start-agent.sh
ExecStop=/bin/bash /root/picluster/systemd/stop-agent.sh
```
#### 2. Modify the start scripts in the systemd folder

For each file that begins with "start", modify the following variables for your installation.
```
export PICLUSTER_AGENT_PATH="/root/picluster/agent"
export PORT="3001"
export AGENTPORT="3002"
```

#### 3. Copy the systemd files to the systemd directory
```
cp systemd/*.service /lib/systemd/system/
```

#### 4. Enable the services

To enable the server service.
```
systemctl enable picluster-server.service
```

To enable the agent service.
```
systemctl enable picluster-agent.service
```

#### 5. Reboot for the services to be started properly
```
Reboot
```
