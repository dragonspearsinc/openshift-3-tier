## Intent

This is a sample project to create and deploy 3 tier application into Kubernetes.  It is NOT production ready and should not be considered to be production code. It is purely to demonstrate the build / deploy of dockerized applications onto a Kubernetes platform.  

This application has only been tested on Windows 10/11.  It may work on other OS's but no guarantee.  

It is composed of 4 parts:

1. [Angular 14 front-end](docker-sample-angular) - See the docker-sample-angular folder
2. [Spring / Maven back-end](springboot-crud) - See the springboot-crud folder
3. [MySQL 8 database server](docker-mysql-8) - See the docker-mysql-8 folder
4. [K8s](k8s) - See the k8s folder.  This is where all of our Kubernetes specific deployment manifests are held


## Dependencies

1. MS Windows
1. OC Cli
1. [Java](https://access.redhat.com/jbossnetwork/restricted/softwareDetail.html?softwareId=104805&product=core.service.openjdk&version=17.0.5&downloadType=distributions) / JAVA_HOME is added to the PATH.
1. Docker Desktop
1. Docker Compose
1. Kubetctl
1. [Workbench](https://www.mysql.com/products/workbench/)
1. Postman
1. NPM

## Login
1. ![Copy Login](support/Copy%20Login%20Command.png)
1. ![Display Token](support/Click%20Display%20Token.png)
1. `oc login --`
 

## Useful Commands
`oc get pods`
`oc get services`
`oc port-forward <POD_NAME> :3306`

## Architecture

Standard 2-tier Spring / MySQL


## First Time Process to run the application (Set-up the DB)

1. Login to the Portal
1. ![Add](support/Add.png)
1. ![Database](support/Click%20Database.png)
1. ![MySQL](support/Add%20MySQL.png)
1. Database Name - `db`
1. MySQL Username - `admin`
1. MySQL Password - `123456`
1. MySQL Root Password - `123456`
1. Next set-up port forwarding so we can configure the db
1. `oc get pods wide -o wide` - make note of the db pod
1. `oc port-forward {pod} 3306:3306`
1. Open up Workbench, point at localhost:3306
1. Create some data
```
CREATE SCHEMA `employee-schema`;

CREATE TABLE `employee-schema`.`employee` (
  `emp_id` INT(11) NOT NULL AUTO_INCREMENT,
  `first_name` VARCHAR(45) NULL,
  `last_name` VARCHAR(45) NULL,
  `email_id` VARCHAR(45) NULL,
  PRIMARY KEY (`emp_id`));

INSERT INTO `employee-schema`.`employee` (`emp_id`, `first_name`, `last_name`, `email_id`) VALUES ('1', 'openshift first 1', 'first 1', 'email 1');
INSERT INTO `employee-schema`.`employee` (`emp_id`, `first_name`, `last_name`, `email_id`) VALUES ('2', 'openshift first 2', 'first 2', 'email 2');
INSERT INTO `employee-schema`.`employee` (`emp_id`, `first_name`, `last_name`, `email_id`) VALUES ('3', 'openshift first 3', 'first 3', 'email 3');
```

## Deploy the App
1. Login to the Portal
1. ![Add](support/Add.png)
1. ![Add Image](support/Add%20Image.png)
1. `docker.io/dragonspears/backend`
1. Select the app `2-tier-app`
1. Create a unique name for the `backend{}`
1. Let the image deploy
1. Try and browse to show that the image is deployed
1. Open Postman to show that we are not getting data
1. ![Edit](support/Edit%20Deployment.png)
1. Scroll to Environment Variables
1. `spring.datasource.username` - `root`
1. `spring.datasource.password` - `123456` //this needs to match the root password that you set before
1. `spring.datasource.url` - `jdbc:mysql://db:3306/employee-schema`  //we are connecting to a service called "db"
--

## Now let's create a server level config so that others can manage

1. Click Create ConfigMaps
1. ![Create ConfigMap](support/Create%20ConfigMap2.png)
1. Click CreateMap
1. ![Create ConfigMap](support/Create%20ConfigMap.png)
1. Create a name, note it... `test-configmap`
1. Call it `spring.datasource.username`
1. Paste in `root`
1. Create another key
1. Call it `spring.datasource.url`
1. Paste in `jdbc:mysql://db:3306/employee-schema`
1. Save


1. `git clone https://github.com/dragonspearsinc/kubernetes-3-tier`
1. `cd kubernetes-3-tier`
1. Build the frontend (this could be done by the CI) - `docker build -t dragonspears/frontend ./docker-sample-angular`
1. Build the frontend (this could be done by the CI) - `docker build -t dragonspears/backend ./springboot-crud`

#
Deploy the app
#
1. Review what will be deployed - `kubectl kustomize ./k8s`
1. `kubectl apply -k k8s` This command is similar to docker compose up and uses a simlar file called [kustomize.yaml](k8s/kustomization.yaml) which creates the pods using Kubernetes specific syntax vs. Docker
    - Note:  To remove the directory based deployment, run `kubectl delete -k k8s`

#
Setup the DB
#
1. `kubectl get pods -o wide` - Review the pods, look at how the IP's are private, non-routable, make note of the db pod name
1. `kubectl exec -it {podname} /bin/bash` 
1. `mysql -p`  //default password is `123456`
1. `update mysql.user set host = '%' where user='admin';`
1. `update mysql.user set host = '%' where user='root';`
1. `exit`  //exit the mysql
1. `exit` //exit the pod / bash
1. `kubectl delete pod {podname}`  //this will kill the pod, but don't worry, since we used a deployment, Kubernetes will automatically restart a new pod
1. `kubectl get pods -o wide` //node that the pod was restarted
1. `kubectl port-forward {podname} 3306:3306` //set-up port-forwarding so we can have access to the DB, otherwise, i have to use mysql commands within the pod itself
1. Open up Workbench, point at localhost:3306
1. Create some data
```
CREATE SCHEMA `employee-schema`;

CREATE TABLE `employee-schema`.`employee` (
  `emp_id` INT(11) NOT NULL AUTO_INCREMENT,
  `first_name` VARCHAR(45) NULL,
  `last_name` VARCHAR(45) NULL,
  `email_id` VARCHAR(45) NULL,
  PRIMARY KEY (`emp_id`));

INSERT INTO `employee-schema`.`employee` (`emp_id`, `first_name`, `last_name`, `email_id`) VALUES ('1', 'openshift first 1', 'first 1', 'email 1');
INSERT INTO `employee-schema`.`employee` (`emp_id`, `first_name`, `last_name`, `email_id`) VALUES ('2', 'openshift first 2', 'first 2', 'email 2');
INSERT INTO `employee-schema`.`employee` (`emp_id`, `first_name`, `last_name`, `email_id`) VALUES ('3', 'openshift first 3', 'first 3', 'email 3');
```
#
Test the deployment
#

Let's connect to the services.  No need to port-forward
1. `kubectl get services -o wide` - Review the services.
1. Open a *browser* to: `http://localhost:8080`

![Browser](./support/Browser%203000.png)
1. Open your *postman* to: `http://localhost:8080`

![Postman](./support/Postman%20Image%208080.png)

## Stop the application

1. `kubectl delete -k k8s`

## Scenarios

# Update the Deployment Replicas

1. Open up `./k8s/frontend-deployment.yaml`
1. Change the `replicas: 1` to `replicas: 4`
1. `kubectl apply -k k8s`
1. `kubectl describe deployment frontend` Review the deployment, show it has changed
1. `kubectl get pods -o wide`

![Replicas](./support/Replicas.png)

# Update the Deployment to use external (outside K8s) DB

** Note:  This section assumes you have a mysql running external to the cluster, configured, and has data

1. Open up `./k8s/backend-deployment.yaml`
1. Change the connection string from `jdbc:mysql://db:3306/employee-schema` to `jdbc:mysql://db:3306/employee-schema`
1. `kubectl apply -k k8s`
1. `kubectl describe deployment backend` Review the deployment, show it has changed
1. Open a browser to see how the data is now changed: http://localhost:3000
