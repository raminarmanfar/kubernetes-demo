# Kubernetes Demo

A dome shows how to use kubernetes with a sample demo node.js app using Docker. There are two ways to manage and interact kubernetes. (a) imparative and (b) declarative. In **imprative** approach we run a command and in affects immediately. However, on the other hand, in **declarative** approach we store all scripts in a file and run all of them in a batch mode (like docker-compose).

## Imprative approach

Common imprative commands to manage kubernetes.
Steps:

1. build your docker image to create a container.
   ```
   docker build -t raminarmanfar/kube-demo-app .
   ```
2. push your container into the docker hub repository.
   ```
   docker push raminarmanfar/kube-demo-app
   ```
3. check if the minikube cluster is up and running.
   ```
   minikube status
   ```
4. if it is stopped, first try to start it using the following script.
   ```
   minikube start --driver=docker
   ```
5. check if kubectl installed and running in your machine.
   ```
   kubectl help
   ```
6. create **deployment** object
   ```
   kubectl create deployment demo-app --image=raminarmanfar/kub-demo-app
   ```
7. get created **deployments** and **pods** objects
   ```
   $ kubectl get deployments
   $ kubectl get pods
   ```

- Note: you can delete any kind of resources using the follwoing command:
  ```
  kubectl delete <resource-type> <resourcename>
  ```
- Sample:
  ```
  kubectl delete deployment demo-app
  ```

8.  start the minikube dashboard in order to see all resources visually.
    ```
    minikube dashboard
    ```
9.  we need a **service** in order to make it possilbe the accessbility to the **pods**. It is because the pods are possible to be stopped and regenerated based on needs as well as on crash. Furthermore, the IP address of pods are only accessible internally not from outside world. therefore, we need to create a service to handle a solid access to the pods.

    ```
    kubectl expose deployment <resource-name> type=<service-type> --port=<app-port-number>
    ```

    - Note: **type** parameter determines the access type of the service and could have the following values.
      - ClusterIP: only accessible inside kubernetes' the cluster
      - NodePort: accessible from internet but without load-balancing
      - LoadBalancer: accessible from the internet with load-balancing mechanism.
    - Sample:

    ```
    kubectl expose deployment demo-app type=LoadBalancer --port=3000
    ```

10. check if the **service** has been created successfully.
    ```
    kubectl get services
    ```
    - Note: you can delete old service using the following command:
    ```
    kubectl delete service <service-name>
    ```
11. to run the app using service, enter the following command:
    ```
    minikube service <service-name>
    ```
12. scale up and down pods manually
    ```
    kubectl scale deployment/demo-app --replicas=3
    ```
    it means that the same pod instance will run 3 times. after scalling you can get pods list to see all running instances. Whenever, any error occurs on any one of the pods, the traffic will be destributed to other healthy pods.
    - Note: it is possible to scale down or up at any time. By scaling down, other pods will be terminated. By scaling up new pods will be generated.
13. update the docker image and container. In order to update the container do the following steps.

    1. build the updated image to create a new container with the latest version of the application with a new tag.

    - Note that if we don't set tag in the new container, it will not be detected by kubernetes.

    2. push the created container into docker hub with a new tag.
    3. set the new updated container into the current deployment object.

    ```
    $ docker build -t raminarmanfar/kub-demo-app:tag-name .

    $ docker push raminarmanfar/kub-demo-app:tag-name

    $ kubectl set image deployment/demo-app kub-demo-app=raminarmanfar/kub-demo-app:tag-name
    ```

    4. check the image update status using the following command:

    ```
    kubectl rollout status deployment/demo-app
    ```

14. undo/rullback the latest deployment
    ```
    kubectl rollout undo deployment/demo-app [--to-revision=revision-number]
    ```
    - Note: parameter **to-revision** is used to undo to a specific revision history. without this parameter it will be rollback to the latest one.
    - Note: you can list the all update history (revisions) of deployment using the following command:
      - parameter **revision** is optional and shows the detail about each update. to see **history-numbers** first run the command without **revision** parameter.
    ```
    kubectl rollout history deployment/demo-app [--revision=history-number]
    ```

## Declerative approach

In **declarative** approach we put all commands in a file and run all commands together. in this approach we should create a **config yaml** file and put all necessary commands in that file. Then run the command to execute all commands inside that config file.

- Note: **config** file can also be a **json** file. However, **yaml** file is more convinient and adviced to be used in kubernetes.
- Note: don't forget to delete all previous objects such as deployments, pods, and services before starting this section.

```
$ kubectl delete deployment demo-app
$ kubectl delete service demo-app
```

### YAML file contents and run it.

To see the detailed yaml file format check [YAML File](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) The yaml file content are as follow:

```
apiVersion: app/v1
kind: Deployment/Service/Job/...
metadata:
    name: second-demo-app-deployment
spec: # specification of deployment
    replicas: 0..n # number of replicas
    selector:
        matchLabels:
            app: second-demo-app # should be the same labels defined in template metadata section. it shows this deployment will manage whitch pods.
    template:
        kind: Pod # this is optional since the kind is already **Deployment** on the second line.
        metadata:
            label:
                app: second-demo-app # you can choose any name for the keys and values of the label
        spec: # specification of the pods
            containers:
                - name: second-node-js-app
                  image: raminarmanfar/kub-demo-app:2
```

To run the scripts to create objects use the following command:

```
kubectl apply -f=<config-file.yaml>
```

In this step, the deployment as well as pods have been created. However, we need to create a service in order to make it available in the internet. We should create another service-config-file.yaml for this purpose. To see the service-config.yaml file check the **service.yaml** in this project. Run the service script file using the following command:

```
$ kubectl apply -f=<service-config-file.yaml>
$ kubectl get services
$ minikube service <service-name>
```

- Note 1: you can make any changes in the config files and run them again using **kubectl apply** command then all changes will be applied automatically to the cluster.
- Note 2: you can still delete all resources using the imprative commands. However, the easier way to delete the resoureces generated by config files you can simply run the following command:

```
kubectl delete -f=<config-file-1.yaml> [-f=config-file-2.yaml ...]
```

OR

```
kubectl delete -f=config-file-1.yaml[,config-file-2.yaml, ...]
```

### Single yaml file for all resources

It is also possible to put all configurations in a single yaml file and run it all once together. Take a look at **master-deployment.yaml** file in this project.

- Note: it is mandatory to use **_---_** seperator between each resource configuration.
