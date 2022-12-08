## Intent

This is a sample project to create and deploy a dockerized 3 tier application.  It is NOT production ready and should not be considered to be production code. It is purely to demonstrate the build / deploy of dockerized applications.  

This application has only been tested on Windows 10/11.  It may work on other OS's but no guarantee.  

It is composed of 3 parts:

1. [Angular 14 front-end](docker-sample-angular) - See the docker-sample-angular folder
2. [Spring / Maven back-end](springboot-crud) - See the springboot-crud folder
3. [MySQL 8 database server](docker-mysql-8) - See the docker-mysql-8 folder
4. [MySQL 8 database server](docker-mysql-8) - See the docker-mysql-8 folder


## Dependencies

1. MS Windows
1. [Java](https://access.redhat.com/jbossnetwork/restricted/softwareDetail.html?softwareId=104805&product=core.service.openjdk&version=17.0.5&downloadType=distributions) / JAVA_HOME is added to the PATH.
1. Docker Desktop
1. Docker Compose
1. Kubetctl
1. [Workbench](https://www.mysql.com/products/workbench/)
1. Postman
1. NPM

## Architecture
[Architecture](assets/architecture.png)

## Troubleshooting commands
1.  Show info about running pod - `kubectl get pods -o wide`
1.  Show configuration about pod - `kubectl describe pod {podname / id}`
1.  Access the pod from your local machine - `kubectl port-forward {podname / id} {outside port}:{container port}` 



## First Time Process to run the application

1. `git clone https://github.com/dragonspearsinc/kubernetes-3-tier`
1. `cd kubernetes-3-tier`
1. Build the frontend (this could be done by the CI) - `docker build -t dragonspears/frontend ./docker-sample-angular`
1. Build the frontend (this could be done by the CI) - `docker build -t dragonspears/backend ./springboot-crud`
1. Review what will be deployed - `kubectl kustomize ./k8s`
1. `kubectl apply -k k8s` This command is similar to docker compose up and uses a simlar file called [kustomize.yaml](k8s/kustomization.yaml) which creates the pods using Kubernetes specific syntax vs. Docker
    - Note:  To remove the directory deployment, run `kubectl delete -k k8s`
1. `kubectl get deployment` - Review what was deployed
1. `kubectl get pods -o wide` - Review the pods
1. `kubectl get services` - Review the services
1. `docker exec -it docker-sample-3-tier-db-1 mysql -uroot -p`  //default passwrd is `123456`
1. `update mysql.user set host = '%' where user='admin';`
1. `update mysql.user set host = '%' where user='root';`
1. `docker restart docker-sample-3-tier-db-1`
1. Open up Workbench
1. [Create Employee Schema](docker-mysql-8/create-schema.sql)
1. [Create Table Called Employee](docker-mysql-8/create-table.sql) 
1. [Create Some Data](docker-mysql-8/create-data.sql) 
1. Open up Postman, do a GET on `http://localhost:8080/api/employees/` -- This tests connectivity to the DB via the Spring/Maven backend.
1. Open up the browser to:  `http://localhost:3000` -- This will show end to end connectivity, no employees, and show an environment variable being used (if it has been created in a `.env` file).  

## Run the application

1. `docker compose up -d`

## Stop the application

1. `docker compose down`
