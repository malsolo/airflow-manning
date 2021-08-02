# Manning's Data Pipelines with Apache Airflow

See https://www.manning.com/books/data-pipelines-with-apache-airflow

## First DAG

Rocket launch DAG from Chapter 2.

After creating (copying and pasting the original source code) we need it to "compile" in the IDE (currently I'm using Visual Studio Code)

For doing so, I use a virtual environment and then I follow the instructions for [installing Apache Airflow locally](https://airflow.apache.org/docs/apache-airflow/stable/start/local.html), that comes with all the necessary libraries.

```
$ virtualenv -p python3.8 venv
$ source venv/bin/activate
$ export PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
$ export AIRFLOW_VERSION=2.1.2
$ CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
$ pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

Aditionally, I need more libraries:
```
$ pip install requests 
```

### Running locally

I will use [Docker for running the DAG](https://airflow.apache.org/docs/apache-airflow/stable/start/docker.html) locally:

Not necessary because the docker-compose.yaml file already exists in the project after executing for the first time:
```
$ curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.1.2/docker-compose.yaml'
```

As explained in the documentation:

> This file contains several service definitions:
> 
> airflow-scheduler - The scheduler monitors all tasks and DAGs, then triggers the task instances once their dependencies are complete.
> 
> airflow-webserver - The webserver available at http://localhost:8080.
> 
> airflow-worker - The worker that executes the tasks given by the scheduler.
> 
> airflow-init - The initialization service.
> 
> flower - The flower app for monitoring the environment. It is available at http://localhost:5555.
> 
> postgres - The database.
> 
> redis - The redis - broker that forwards messages from scheduler to worker.
> 
> All these services allow you to run Airflow with CeleryExecutor. 

For more information, see [Architecture Overview](https://airflow.apache.org/docs/apache-airflow/stable/concepts/overview.html).

Then, the directories dag, logs and plugin should exist (unnecesary again):
```
$ mkdir ./logs ./plugins
```

And we also need user permissions (this is necessary because the .env file is _git ignored_)
```
$ echo -e "AIRFLOW_UID=$(id -u)\nAIRFLOW_GID=0" > .env
```

Now run the database migration:
```
$ docker-compose up airflow-init

airflow-init_1       | Admin user airflow created
airflow-init_1       | 2.1.2
rocket_airflow-init_1 exited with code 0
```

Now you can start all services:

```
$ docker-compose up
```

The webserver available at: http://localhost:8080. The default account has the login airflow and the password airflow.

Recommended for [Running the CLI commands](https://airflow.apache.org/docs/apache-airflow/stable/start/docker.html#running-the-cli-commands)

Unnecesary download:
```
$ curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.1.2/airflow.sh'
$ chmod +x airflow.sh
```

Now you can run commands easier:
```
$ ./airflow.sh info
$ ./airflow.sh bash
$ ./airflow.sh python
```

**Note:** docker-compose.yaml modified for not loading examples.

Go to localhost:8080, airflow/airflow, run the dag, and then, login into a worker node to see the download images:

```
$ docker ps        CONTAINER ID   IMAGE                  COMMAND                  CREATED             STATUS                    PORTS                                                 NAMES
e4259cca7d0c   apache/airflow:2.1.2   "/usr/bin/dumb-init …"   6 minutes ago       Up 6 minutes (healthy)    0.0.0.0:5555->5555/tcp, :::5555->5555/tcp, 8080/tcp   rocket_flower_1
8981bfbe18c9   apache/airflow:2.1.2   "/usr/bin/dumb-init …"   6 minutes ago       Up 6 minutes (healthy)    8080/tcp                                              rocket_airflow-scheduler_1
cb035299eea7   apache/airflow:2.1.2   "/usr/bin/dumb-init …"   6 minutes ago       Up 6 minutes (healthy)    0.0.0.0:8080->8080/tcp, :::8080->8080/tcp             rocket_airflow-webserver_1
3d8e8b93a85d   apache/airflow:2.1.2   "/usr/bin/dumb-init …"   6 minutes ago       Up 6 minutes (healthy)    8080/tcp                                              rocket_airflow-worker_1
d6fc2cc1b708   postgres:13            "docker-entrypoint.s…"   About an hour ago   Up 57 minutes (healthy)   5432/tcp                                              rocket_postgres_1
6450afd6e6bd   redis:latest           "docker-entrypoint.s…"   About an hour ago   Up 57 minutes (healthy)   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp             rocket_redis_1

$ ❯ docker exec -it 3d8e8b93a85d /bin/bash
default@3d8e8b93a85d:/opt/airflow$ ls /tmp/images/
antares2520230252b_image_20191102024633.jpeg
atlas2520v2520n22_image_20190224012241.jpeg
falcon2520925_image_20210619160237.png
falcon2520925_image_20210619160353.png
gslv2520mk2520ii_image_20190825171642.jpg
hyperbola-1_image_20190724014013.jpg
long2520march25203_image_20190224025008.jpeg
soyuz25202.1b_image_20210520085931.jpeg
soyuz_image_20190222031122.jpeg
vega_image_20201111143622.jpeg
default@3d8e8b93a85d:/opt/airflow$ 
```

