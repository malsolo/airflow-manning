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

