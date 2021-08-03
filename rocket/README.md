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

## Kubernetes

TODO: document installation of
- Kubernetes:
    - minikube, https://minikube.sigs.k8s.io/docs/start/ and https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/ 
    - kubectl, https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/#install-with-homebrew-on-macos
    - TODO: kind, https://kind.sigs.k8s.io/docs/user/quick-start
- Helm: https://helm.sh/docs/intro/quickstart/
- Kind: https://kind.sigs.k8s.io/docs/user/quick-start (just brew install kind)

- Helm chart:

https://artifacthub.io/packages/helm/apache-airflow/airflow

### Helm Chart

Documented at https://airflow.apache.org/docs/helm-chart/stable/index.html

```
$ kubectl create namespace airflow

$ helm repo add apache-airflow https://airflow.apache.org

$ helm install airflow apache-airflow/airflow --namespace airflow
NAME: airflow
LAST DEPLOYED: Mon Aug  2 20:22:31 2021
NAMESPACE: airflow
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing Apache Airflow 2.1.2!

Your release is named airflow.
You can now access your dashboard(s) by executing the following command(s) and visiting the corresponding port at localhost in your browser:

Airflow Webserver:     kubectl port-forward svc/airflow-webserver 8080:8080 --namespace airflow
Flower dashboard:      kubectl port-forward svc/airflow-flower 5555:5555 --namespace airflow
Default Webserver (Airflow UI) Login credentials:
    username: admin
    password: admin
Default Postgres connection credentials:
    username: postgres
    password: postgres
    port: 5432

You can get Fernet Key value by running the following:

    echo Fernet Key: $(kubectl get secret --namespace airflow airflow-fernet-key -o jsonpath="{.data.fernet-key}" | base64 --decode)

$ helm list --namespace airflow
NAME   	NAMESPACE	REVISION	UPDATED                              	STATUS  	CHART        	APP VERSION
airflow	airflow  	1       	2021-08-02 20:22:31.270734 +0200 CEST	deployed	airflow-1.1.0	2.1.2

```

Now, for accesing the application, we have to do a port forward:

```
$ kubectl get svc -n airflow
NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
airflow-flower                ClusterIP   10.105.221.230   <none>        5555/TCP            10h
airflow-postgresql            ClusterIP   10.104.11.136    <none>        5432/TCP            10h
airflow-postgresql-headless   ClusterIP   None             <none>        5432/TCP            10h
airflow-redis                 ClusterIP   10.98.137.233    <none>        6379/TCP            10h
airflow-statsd                ClusterIP   10.109.76.135    <none>        9125/UDP,9102/TCP   10h
airflow-webserver             ClusterIP   10.103.232.238   <none>        8080/TCP            10h
airflow-worker                ClusterIP   None             <none>        8793/TCP            10h

$ kubectl port-forward svc/airflow-webserver 8080:8080 --namespace airflow
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080

```

😄

### Extending Airflow Image

See https://airflow.apache.org/docs/helm-chart/stable/quick-start.html#extending-airflow-image

Note, instructions above for deploying Airflow with the official Helm Chart for Apache Airflow.

Let's review the Python version

See https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/ 

```
$ kubectl get pods -n airflow
NAME                                 READY   STATUS    RESTARTS   AGE
airflow-flower-6d49cc7fd4-njjgh      1/1     Running   0          11h
airflow-postgresql-0                 1/1     Running   0          11h
airflow-redis-0                      1/1     Running   0          11h
airflow-scheduler-5bc486d97f-8flz2   2/2     Running   0          11h
airflow-statsd-7586f9998-krsk4       1/1     Running   0          11h
airflow-webserver-66fd4f75b9-mlqft   1/1     Running   0          11h
airflow-worker-0                     2/2     Running   0          11h

$ kubectl get po airflow-webserver-66fd4f75b9-mlqft -n airflow
NAME                                 READY   STATUS    RESTARTS   AGE
airflow-webserver-66fd4f75b9-mlqft   1/1     Running   0          11h

$ kubectl exec --stdin --tty airflow-webserver-66fd4f75b9-mlqft -n airflow -- /bin/bash
Defaulted container "webserver" out of: webserver, wait-for-airflow-migrations (init)
airflow@airflow-webserver-66fd4f75b9-mlqft:/opt/airflow$ python -V
Python 3.6.13
```

Now let's create the image but in the minikube cluster Docker daemon:

https://minikube.sigs.k8s.io/docs/handbook/pushing/#1-pushing-directly-to-the-in-cluster-docker-daemon-docker-env

```
$ eval $(minikube docker-env)
```

Note, for unsetting: https://minikube.sigs.k8s.io/docs/commands/docker-env/ (options: -u, --unset          Unset variables instead of setting them)

```
$ eval $(minikube docker-env -u)
```

*To verify your terminal is using minikube’s docker-env you can check the value of the environment variable MINIKUBE_ACTIVE_DOCKERD to reflect the cluster name.*


For creating the image see:
* https://hub.docker.com/layers/apache/airflow/2.1.2/images/sha256-7709f9fb1d679f1874fe7df4902e347025bf17a3f82b2af8fddfc1962ecbb412?context=explore
* https://airflow.apache.org/docs/helm-chart/stable/manage-dags-files.html#bake-dags-in-docker-image

From the Dockerfile we have:

```
$ eval $(minikube docker-env)

$ docker build --tag rocket-airflow:1.0.0 .

$ docker images
REPOSITORY                                TAG                                          IMAGE ID       CREATED          SIZE
rocket-airflow                            1.0.0                                        240647d1512a   11 seconds ago   888MB
...

$ helm upgrade --install airflow apache-airflow/airflow \
  --set images.airflow.repository=rocket-airflow \
  --set images.airflow.tag=1.0.0 \
  --namespace airflow
Release "airflow" has been upgraded. Happy Helming!
NAME: airflow
LAST DEPLOYED: Tue Aug  3 08:05:45 2021
NAMESPACE: airflow
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
Thank you for installing Apache Airflow 2.1.2!

Your release is named airflow.
You can now access your dashboard(s) by executing the following command(s) and visiting the corresponding port at localhost in your browser:

Airflow Webserver:     kubectl port-forward svc/airflow-webserver 8080:8080 --namespace airflow
Flower dashboard:      kubectl port-forward svc/airflow-flower 5555:5555 --namespace airflow
Default Webserver (Airflow UI) Login credentials:
    username: admin
    password: admin
Default Postgres connection credentials:
    username: postgres
    password: postgres
    port: 5432

You can get Fernet Key value by running the following:

    echo Fernet Key: $(kubectl get secret --namespace airflow airflow-fernet-key -o jsonpath="{.data.fernet-key}" | base64 --decode)

$ helm list -n airflow
NAME   	NAMESPACE	REVISION	UPDATED                              	STATUS  	CHART        	APP VERSION
airflow	airflow  	2       	2021-08-03 08:05:45.482425 +0200 CEST	deployed	airflow-1.1.0	2.1.2

$ kubectl port-forward svc/airflow-webserver 8080:8080 --namespace airflow

$ eval $(minikube docker-env -u)
```

#### Kubernetes executor

```
$ helm show values apache-airflow/airflow > helm_airflow_values.yml
```

Then, customize values as explained at https://airflow.apache.org/docs/helm-chart/stable/parameters-ref.html#images


