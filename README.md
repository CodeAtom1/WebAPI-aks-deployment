# WebAPI-aks-deployment

1. Create acr manually from portal
2. Create service principal(it's an identity) 
 	- to be used in aks , since aks will have to create nodes in azure it needs to have a way to authenticate using an identity.
3. Go to ACR , grant access to this service principal(SP)
	- So anyone having the SP can authencticate using this SP.
4. Create aks 
 	- with lowest number of nodes & node configuration for dev/testing
 	- Disable monitoring
5. Login to acr: - az acr login --name acrdemo0181
 	- It's going to authenticate our docker engine against container registry
6. View docker images using command:  `docker images`
7. Select and retag your image(maybe including repository name) which makes sense for acr.
 	- docker tag docker-demo:v1 acrdemo0181.azurecr.io/docker-demo:v1
8. Run Docker images command again to see your updated image
9. Run Docker push command with your new image
 	- docker push acrdemo0181.azurecr.io/docker-demo:v1
10. Verify your pushed image in you container registry> repositories
11. Now, Install Kubernetes on your vs code
	- It helps creating yaml files 
12. Create deployment.yml file in root of your project(this will have the deployment of your container/pod and required  settings & flow for pod/aks)
	- What is POD? POD is nothing more than a group of one or more containers that share storage/network. And they 
    also contain the specification of how to run the containers.
13. Deployment.yml
  
```
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: aks-demo-deployment #aks deployment name
	spec:
	  selector:
	    matchLabels:
	      app: aks-demo-pod #pod labels which will have this deployment
	  template:
	    metadata:
	      labels:
	        app: aks-demo-pod #label of current pod
	    spec:
	      containers:
	      - name: aks-demo-container
	        image: acrdemo0181.azurecr.io/docker-demo:v1
	        resources:
	          limits:
	            memory: "128Mi"
	            cpu: "500m"
	        ports:
	        - containerPort: <Port>
```

14. Creating this deployment file will allows us to create the POD. But we have to do a bit more here. This enough will not allow us to 
  connect to our container to query from outside. 
15. To enable this kind of window into POD we have to define service.
	- What is service? Service is a logical representation/object that allows for the interaction between different objects in kubernetes, 
and also potentially allows external communication with the PODs
16. Service.yml
```
	apiVersion: v1
	kind: Service
	metadata:
	  name: aks-demo-service #aks service 
	spec:
	  selector:
	    app: aks-demo-pod #PODs with which this service will connect to
	  ports:
	  - port: 5036 #external port
	    targetPort: 5036 #internal port<PODs port>
	  type: LoadBalancer #kind of service that aquires public ip from cloud hosting provier 
	  	#and assigns that ip to the service so that external 
    		#callers can use that service and communicates with the PODs behind the service
```
 17. Now Connect to aks run following command:-
18. `az aks get-credentials --resource-group aks-demo --name aks-demo-01`
	- This will download the secrets to connect to aks into your box, and it stores it into a config file.
19. You can test the aks connection using below command:-
	- `kubectl get nodes`
		i. This will show current nodes in your aks
		ii. (Kubectl command will be installed during your docker desktop installation.)
20. Apply your deployment.yml file and add nodes
	- `kubectl apply -f .\deployment.yml`
21. Get deployment status of your deployment
	- `kubectl get deployment`
22. Get pod deployment status
	- `kubectl get pods`
23. Now, also deploy the service:-
	- `kubectl apply -f .\service.yml`
	- Cluster IP is kind of internal IP
24. Now verify if your service is up & running using command:-
    `kubectl get services`
