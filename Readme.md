# Kubernetes Demo

A dome shows how to use kubernetes with a sample demo node.js app using Docker.

## Imprative commands

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

8. start the minikube dashboard in order to see all resources visually.
   ```
   minikube dashboard
   ```
9. we need a **service** in order to make it possilbe the accessbility to the **pods**. It is because the pods are possible to be stopped and regenerated based on needs as well as on crash. Furthermore, the IP address of pods are only accessible internally not from outside world. therefore, we need to create a service to handle a solid access to the pods.

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
