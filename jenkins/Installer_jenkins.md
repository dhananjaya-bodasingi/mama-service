Installed Docker
Installed Docker using Homebrew with the command:

brew install --cask docker
Verified the installation:

docker version
Ensured Docker Desktop was installed and running:

open /Applications/Docker.app

Checked Docker context:

docker context ls 
Issues Faced:
Docker daemon wasn’t running initially. This was resolved by starting Docker Desktop manually.

Installed Minikube
Reinstalled Minikube using Homebrew:
brew reinstall minikube
Started Minikube using the Docker driver:
minikube start --driver=docker
Verified Minikube status:
minikube status
Issues Faced:
The Docker daemon needed to be active for Minikube to start successfully. This was resolved by starting Docker Desktop.

Deployed Jenkins on Kubernetes
Created a Jenkins Deployment using a YAML file or Helm chart (details depend on your specific configuration).
Deployed Jenkins in a namespace (e.g., jenkins) using kubectl:
kubectl apply -f <jenkins-deployment.yaml>
Checked the status of the Jenkins pod:
kubectl get pods -n jenkins
Verified the logs of the Jenkins pod to ensure it started properly:
kubectl logs <jenkins-pod-name> -n jenkins
Key Log Output: The Jenkins pod logs indicated:
Jenkins successfully extracted and started.
A random admin password was generated:
Jenkins initial setup is required. An admin user has been created and a password generated.
Password: 16755f55a5d447e39258ad4cda2e9ace
This password is also stored at:

/var/jenkins_home/secrets/initialAdminPassword

Accessed Jenkins
Exposed the Jenkins service:
If using a Kubernetes LoadBalancer:

kubectl expose deployment jenkins --type=LoadBalancer --name=jenkins-service -n jenkins
If using Minikube:

minikube service <jenkins-service-name> -n jenkins

Accessed Jenkins in a browser at the provided URL (e.g., http://<node-ip>:8080).

Initial Jenkins Setup
Entered the admin password from the logs or retrieved it:

kubectl exec <jenkins-pod-name> -n jenkins -- cat /var/jenkins_home/secrets/initialAdminPassword

Followed the setup wizard to:
Install recommended plugins.
Configure an admin user.
Set up Jenkins for use.

Deployed Jenkins
Created a namespace for Jenkins (if applicable):
kubectl create namespace jenkins
Applied Jenkins deployment and service YAML:
kubectl apply -f <jenkins-deployment-file>.yaml -n jenkins
Confirmed the resources are running:
kubectl get all -n jenkins
Output showed the jenkins-master pod Running, a LoadBalancer service, and associated deployment/replicaset.

Checked External Access
The LoadBalancer service was created, but its external IP was <pending> because Minikube doesn't support LoadBalancer services natively.
Attempted to access Jenkins using the NodePort provided by Minikube but faced connectivity issues:
curl -I http://<minikube-node-ip>:<NodePort>

Used Port Forwarding for Access
Set up port forwarding to access Jenkins on localhost:
kubectl port-forward service/jenkins-master 8080:8080 -n jenkins
Accessed Jenkins at http://localhost:8080.

Verified Kubernetes Cluster Status
Confirmed Minikube node and cluster status:
kubectl get nodes -o wide
kubectl cluster-info
Debugged Java Issues in Pods
Accessed the jenkins-master pod:
kubectl exec -it jenkins-master-<id> -n jenkins -- /bin/bash

Checked the Java version inside the pod:
java -version

Networking Debugging
Verified if Jenkins service port (NodePort) was open on the host:
netstat -tuln | grep <NodePort>
Tested connectivity using curl to the Jenkins URL.

Current Problem
Jenkins occasionally loads but becomes unreachable, likely due to:
Misconfiguration of the Minikube network.
Resource limits or pod crashes.
LoadBalancer service not being supported by Minikube.

Next Steps
To stabilize and fully access Jenkins:
Switch to NodePort Service: Update the service YAML to use type: NodePort instead of LoadBalancer.: 

kind: Service
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 32000  # Ensure this is within the NodePort range (30000–32767)

Apply the updated file:
kubectl apply -f <service-file>.yaml

Access Jenkins via Minikube NodePort:

minikube ip
Access Jenkins at http://<minikube-ip>:<NodePort>.
Monitor Resource Usage:

kubectl top pods -n jenkins
kubectl describe pod jenkins-master-<id> -n jenkins
Test Stability: Confirm the service doesn't drop connections.
