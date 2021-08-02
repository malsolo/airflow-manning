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




