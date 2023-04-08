# node-js-deployment-kubernetes
Deploying Node.js Application In Kubernetes

# step: 1 creat a docker file for your application and build the image

***example Dockerfile for the Node.js application:***

```
# Specify the base image
FROM node:14

# Create and set the working directory in the container
WORKDIR /usr/src/app

# Copy package.json and package-lock.json files to the container
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application files to the container
COPY . .

# Expose port 3000
EXPOSE 3000

# Start the application
CMD ["npm", "start"]
```
***place this file in the root directory of your Node.js application. Then, use the following commands to build and run the Docker image:***

```
# Build the Docker image
docker build -t my-node-app .

# Run the Docker container
docker run -p 3000:3000 my-node-app
```
***This will build a Docker image called "my-node-app" and start a container based on that image. The container will be accessible at http://localhost:3000***

# step: 2  push your node-js image in docker hub

***here are the steps to push a Docker image to Docker Hub:***

1: Login to Docker Hub from the command line using the docker login command. You'll need to enter your Docker Hub username and password.

```
docker login
```
2: Tag the Docker image using the following format:

```
docker tag IMAGE_NAME USERNAME/REPOSITORY:TAG
```
***For example:***

```
docker tag my-node-app myusername/my-node-app:latest
```
This will tag the my-node-app image as myusername/my-node-app:latest, which is the format required by Docker Hub.

3: Push the tagged image to Docker Hub using the docker push command:

```
docker push USERNAME/REPOSITORY:TAG
```
***For example:***

```
docker push myusername/my-node-app:latest
```

This will upload the my-node-app image to Docker Hub under the myusername/my-node-app repository with the latest tag.

That's it! Your Docker image should now be available on Docker Hub.

# step: 3 creat Kubernetes deployment YAML file

Here is an example Kubernetes deployment YAML file that you can use to deploy your Node.js application:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-node-app
  labels:
    app: my-node-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-node-app
  template:
    metadata:
      labels:
        app: my-node-app
    spec:
      containers:
      - name: my-node-app
        image: myusername/my-node-app:latest # Replace with your Docker image
        ports:
        - containerPort: 3000
```

You'll need to replace myusername/my-node-app:latest with the name of your Docker image on Docker Hub.

Save this file as deployment.yaml and run the following command to deploy it to your Kubernetes cluster:

```
kubectl apply -f deployment.yaml
```
This will create a deployment with one replica of your Node.js application running in a container.

# step: 4 create a Kubernetes service.yaml file

If you also want to expose the application to the outside world, you'll need to create a Kubernetes service for it. Here is an example YAML file for a NodePort service:

```
apiVersion: v1
kind: Service
metadata:
  name: my-node-app-service
spec:
  type: NodePort
  selector:
    app: my-node-app
  ports:
  - name: http
    port: 80
    targetPort: 3000
    nodePort: 30001
```
Save this file as service.yaml and run the following command to create the service:

```
kubectl apply -f service.yaml
```
This will create a NodePort service that maps port 30001 on the host machine to port 3000 in the container running your Node.js application.

You should now be able to access your application by visiting the IP address of your Kubernetes cluster and the NodePort you specified in the service YAML file, like so: http://<cluster-ip>:30001.

***also you can create the service with loadbalancer***

Here's an example YAML file for a Kubernetes service with a LoadBalancer:

```
apiVersion: v1
kind: Service
metadata:
  name: my-node-app-service
spec:
  type: LoadBalancer
  selector:
    app: my-node-app
  ports:
  - name: http
    port: 80
    targetPort: 3000
```

This will create a LoadBalancer service that maps port 80 to port 3000 in the container running your Node.js application.

Once the service is created, Kubernetes will allocate an external IP address for the LoadBalancer. You can use this IP address to access your application from outside of the Kubernetes cluster.

***To view the external IP address of the LoadBalancer, you can run the following command:***

```
kubectl get services my-node-app-service
```
This will display the external IP address of the LoadBalancer in the EXTERNAL-IP column.

You should now be able to access your application by visiting the external IP address of the LoadBalancer in your web browser.




