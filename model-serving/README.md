This tutorial will cover the following two topics:

1. Dynamic contextualization and model serving in a scalable production environment 
3. Reliable continues integration and development process

--------------------

Important:

_It is expected that you have finished the prelab of the distributed e-infrastructures part._

_Please terminate VMs once you finish the task._ 

0. The tasks and the configurations are designed for the Ubuntu 18.04 with medium VM flavour.  

1. We will use SNIC Science Cloud (SSC) for all the tasks. In case you want to learn more about SSC visit website  http://cloud.snic.se

2. We recommend you to create a VM in SSC and execute all the tasks on that virtual machine. You can run the tasks on your laptops but it may break your local working environment.

3. For all these tasks clone the repository:

https://github.com/SNICScienceCloud/technical-training.git

The code for all the tasks is available in the "model-serving" directory. 

```
 - model-serving
   -- Single_server_without_docker
   -- Single_server_with_docker
   -- CI_CD
   -- OpenStack-Client
```
------------------------

# Dynamic contextualization

Dynamic contextualization is a process of preparing a customized computing environment at runtime. The process ranging from creating/defining user roles and permissions to updating/installing different packages. 

## Task-1: Single server deployment 

In this task, we will learn how the dynamic contextualization works using Cloudinit package. For this task, we need to use OpenStack APIs to start a VM and contextualize it at run time. The contextualization process will setup the following working environment: 


1. Flask web application as a frontend server 
2. Celery and RabbitMQ server for backend server
3. Model execution environment based on Keras and TensorFlow

In case you are not familiar with the above-mentioned packages please read the following links: 

1. Flask Application -> https://flask.palletsprojects.com/en/1.1.x/
2. Celery and RabbitMQ -> https://docs.celeryproject.org/en/stable/getting-started/
3. Keras and TensorFlow -> https://www.tensorflow.org/guide/keras
4. OpenStack -> https://www.openstack.org/

A client will send a prediction request from the front-end web server, the server will pass the request to the backend Celery environment where running workers in the setup will pick up the task, run the predictions by loading the available model, send back the results and finally frontend server will display the results.

## Steps for the contextualization

1. Goto `technical-training/model-serving/single_server_without_docker/production_server/`  directory. This directory contains the code that will run on your production server. Following is the structure of the code: 

``` 
 - Flask Application based frontend 
    -- app.py
    -- static
    -- templates
 - Celery and RabbitMQ setup
    -- run_task.py
    -- workerA.py
 - Machine learning Model and Data 
    -- model.h5
    -- model.json
    -- pima-indians-diabetes.csv
```

Open different files and try to understand the application's structure. 

2. Goto the `technical-training/model-serving/openstack-client/single_node_without_docker_client/` directory. This is the code that we will run to contextualize our production server. The code is based on the following two files:

```console
cloud-cfg.txt
start_instance.py
```

Open the files and try to understand the steps. _You need to setup variable values in the start_instance.py script._ 

-------------
In order to run this code, you need to have OpenStack API environment running. Follow the instructions available on the following links: 

i. Goto https://docs.openstack.org/install-guide/environment-packages-ubuntu.html , http://docs.openstack.org/cli-reference/common/cli_install_openstack_command_line_clients.html and download the client tools and API for OpenStack.

ii. Download the Runtime Configuration (RC) file from the SSC site (Project->Compute->Access & Security->API Access->Download OpenStack RC File).

iii. Confirm that your RC file have following enviroment variables:

```console
export OS_USER_DOMAIN_NAME="snic"
export OS_IDENTITY_API_VERSION="3"
export OS_PROJECT_DOMAIN_NAME="snic"
```
iv. Set the environment variables by sourcing the RC-file:

```console 
source <project_name>_openrc.sh
```

v. The successful output of the following commands will confirm that you have the correct packages available on your VM:

```console
openstack server list
openstack image list
```

vi. For the API communication we need following extra packages:

```console
apt install python3-openstackclient
apt install python-novaclient
apt install python-keystoneclient
```

------------------

3. Once you have setup the environment, run the following command. 

```console 
python3 start_instance.py
```

The command will start a new server and initiate the contextualization process. It will take approximately 10 to 15 minutes. The progress can be seen on the cloud dashboard. Once the process finish, attach a floating IP to your production server and access the webpage from your client machine. 

Welcome page `http://<PRODUCTION-SERVER-IP>:5100`. Predictions page `http://<PRODUCTION-SERVER-IP>:5100/predictions`

## Questions

1. Explain how the application works? Please keep your answer brief to one paragraph. 

2. What are the drawbacks of the current deployment strategy adopted in the task-1? Write at least four drawbacks.

3. Currently, the contextualization process takes 10 to 15 minutes. How can we reduce the deployment time?   

-----------------------------

TERMINATE THE SERVER VM STARTED FOR THE TASK-1!

-----------------------------
## Task-2: Single server deployment with Docker Containers

In this task, we will repeat the same deployment but with Docker containers. It will create a flexible containerized deployment environment where each container has a defined role. 

1. Goto `technical-training/model-serving/single_server_with_docker/production_server/` directory. This directory contains the code that will run on your production server. Following is the structure of the code: 

``` 
 - Flask Application based frontend 
    -- app.py
    -- static
    -- templates
 - Celery and RabbitMQ setup
    -- run_task.py
    -- workerA.py
 - Machine learning Model and Data 
    -- model.h5
    -- model.json
    -- pima-indians-diabetes.csv
 - Docker files
    -- Dockerfile
    -- docker-compose.yml
```

Open different files and try to understand the application's structure. 

2 - Goto the `technical-training/model-serving/openstack-client/single_node_with_docker_client/` directory. This is the code that we will run to contextualize the production server. The code is based on the following two files:

```console
cloud-cfg.txt
start_instance.py
```

Open the files and try to understand the steps. _You need to setup variable values in the start_instance.py script._ 

3. Run the following command. 

```console 
python3 start_instance.py
```

The command will start a new server and initiate the contextualization process. It will take approximately 10 to 15 minutes. The progress can be seen on the cloud dashboard. Once the process finish, attach a floating IP to your production server and access the webpage from your client machine. 

Welcome page `http://<PRODUCTION-SERVER-IP>:5100`. Predictions page `http://<PRODUCTION-SERVER-IP>:5100/predictions`

4. The next step is to test the horizontal scalability of the setup. 

4a. Login to the production server. 

```console 
ssh -i ubuntu@<PRODUCTION-SERVER-IP>
```

4b. Switch to the superuser mode.
   
```console
sudo bash
```

4c. Check the cluster status:

```console 
cd /technical-training/model-serving/single_server_with_docker/production_server
docker-compose ps
```

The output will be as following:

```console
   Name                          Command               State                                             Ports                                           
------------------------------------------------------------------------------------------------------------------------------------------------------------------
production_server_rabbit_1     docker-entrypoint.sh rabbi ...   Up      15671/tcp, 0.0.0.0:15672->15672/tcp, 25672/tcp, 4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp
production_server_web_1        python ./app.py --host=0.0.0.0   Up      0.0.0.0:5100->5100/tcp
production_server_worker_1_1   celery -A workerA worker - ...   Up  
```

Following three containers are running on the production server: 

```
production_server_rabbit_1  -> RabbitMQ server
production_server_web_1 -> Flask based web application
production_server_worker_1_1 -> Celery worker 
```

Now we will add multiple workers in the cluster using docker commands. 

4d. Currently, there is one worker available. We will add two more workers. 

```console 
docker-compose up --scale worker_1=3 -d
```

4e. Check the cluster status.

```console
docker-compose ps
```

Output will be:

```console
            Name                          Command               State                                             Ports                                           
------------------------------------------------------------------------------------------------------------------------------------------------------------------
production_server_rabbit_1     docker-entrypoint.sh rabbi ...   Up      15671/tcp, 0.0.0.0:15672->15672/tcp, 25672/tcp, 4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp
production_server_web_1        python ./app.py --host=0.0.0.0   Up      0.0.0.0:5100->5100/tcp
production_server_worker_1_1   celery -A workerA worker - ...   Up
production_server_worker_1_2   celery -A workerA worker - ...   Up                                                             
production_server_worker_1_3   celery -A workerA worker - ...   Up
```

Now we have 3 workers running in the system. 

4f. Now scale down the cluster.

```console 
docker-compose up --scale worker_1=1 -d
```

## Questions

1. What problems Task-2 deployment strategy solves compared to the strategy adopted in Task-1? 

2. What are the outstanding issues that the deployment strategy of Task-2 cannot not address? 

3. What is the difference between horizontal and vertical scalability? The strategy discussed in Task-2, is it horizontal or vertical scalability? 

-----------------------------

TERMINATE THE SERVER VM STARTED FOR THE TASK-2!

-----------------------------


-----------------------------
