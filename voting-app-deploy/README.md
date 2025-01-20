# Voting App Deployment instructions

A lightweight and scalable voting application designed for modern environments. It is deployed on Kubernetes using both pods and deployment files, allowing for flexibility in how the app is managed and scaled.

---
voting-app-Pod-Based Deployment Instructions:

#make 2 node for the cluster one is master and the other is the worker
minikube start --nodes 2

#1st node for deploying the core app pods on it using label
kubectl label nodes #node1-name job=app 

#2nd node for deploying the database pods on it
kubectl label nodes #node1-name job=db

#check nodes
kubectl describe nodes node-name

#get the ip for nodes
kubectl get node -o  wide
#to get the labels
kubectl get nodes --show-labels

kubectl get ns

#run the prod-ns.yaml to create new namespace
kubectl create -f prod-ns.yaml
kubectl get ns

#optionally>> if u want to make the ns is the default
kubectl config set-context --current --namespace=prod

#applying the resourcequota
kubectl apply -f resource-quota.yaml #--namespace=prod >> uou dont have to specify the ns i already configure it in the yaml file

#start deploying the pods
#1- voting app and its service
kubectl create -f voting-app-deploy.yaml
kubectl create -f voting-app-svc.yaml

#2- redis and its service
kubectl create -f redis-deploy.yaml
kubectl create -f redis-svc.yaml

#1- worker doesnt have a service
kubectl create -f worker-app.yaml


#1- database and its secret and service
#creating the secret first
kubectl create -f secret-db.yaml
# i used base64 to Encrypt the username and password >> $echo "postgress" | base64

kubectl create -f postgres-deploy.yaml
kubectl create -f postgres-svc.yaml

#1- result app and its service
kubectl create -f result-app-deploy.yaml
kubectl create -f result-app-svc.yaml

#now check everything is ok
kubectl get node -o wide
kubectl get ns
kubectl get pod,svc #-n prod >> use this option if you dont make the prod the default ns
kubectl get resourcequota #-n prod 
kubectl get sectret -o wide #-n prod 
#to display and ensure the env variables in the postgres pod
kubectl exec -n prod postgres-pod --printenv

#to access the voting app
minikube service -n prod voting-svc --url

#to access the result app
minikube service -n prod result-svc --url

#if you finished you can delete everything now
kubectl delete -f .

