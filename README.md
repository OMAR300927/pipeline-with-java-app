Java demo project: CI/CD pipeline for deploying java application and monitor it

Build and deploy a Java application using Docker, Jenkins, Trivy, SonarQube, Kubernetes, ArgoCD and ( Prometheus + Grafana + BlackBox ) outside the cluster

# Prerequisites

* Java application
* Git and GitHub
* Docker
* Trivy
* SonarQube
* Jenkins
* Kubernetes
* ArgoCD
* Prometheus + Grafana + BlackBox

# Steps in the CI/CD pipeline

1. Get the Java application
2. Dockerize the Java application
3. Push the Docker file to the repository
4. Start Jenkins server on a host
5. Start SonarQube container and make a token
6. Use the token to make credential in Jenkins and add SonarQube server
7. Write Jenkins file
8. Set up Kubernetes on a host using Minikube
9. Create a Kubernetes deployment and service for the Java application
10. Create a Kubernetes deployment and service for postgres as DB for the java application
11. Create a Kubernetes ingress to expose the Java service externally
12. Use Jenkins to deploy the Kubernetes cluster
13. Set an email notification at the end of the pipeline
14. Use ArgoCD and connect it with the repository
15. Use BlackBox to monitor the application in Prometheus
16. Create a dashboard in Grafana

# The pipeline steps

Inside Jenkins file:
Checkout ---> maven clean ---> maven compile ---> maven test ---> SonarQube analysis ---> Quality gate ---> Scan file system ---> maven package ---> build & tag image ---> Scan the image ---> push the image ---> Apply kubernetes cluster --->  post block for the email notification

Outside Jenkins file:
Use ArgoCD and Monitor the application using Prometheus + Grafana + BlackBox

# Project structure

| File                  | Description
|-----------------------|--------------------------------------------------------------------------------------------------------------------------------|
| App.java              |  Java application                                                                                                              |
| AppTest.java          |  Test file for Java application                                                                                                |
| Dockerfile            |  Contains commands to build and run the Docker image                                                                           |
| Jenkinsfile           |  Contains the pipeline script                                                                                                  |
| pom.xml               |  defines the project's dependencies, build configuration, and metadata used by Maven to manage and build the Java application. |
| java.yaml             |  Kubernetes deployment and service for Java application                                                                        |
| postgres.yaml         |  Kubernetes deployment and service for postgres                                                                                |
| ingress.yaml          |  Kubernetes ingress to expose the java service externally                                                                      |
| .gitignore            |  avoid tracking unnecessary or sensitive data in the repository.                                                               |

# Conclusion

* Don't forget to use ngrok in terminal and keep the terminal open so you can use the webhook between Jenkins and SonarQube
* I used credentials for Docker Hub. After deploy the application, run `minikube service html-service` to access it in browser
* Or you can use the tunnel by running `minikube tunnel` to access it , But if you use this method, you should also update your hosts file

# Notes

* In the K8s stage jenkins was using the credential from the temporary path, but inside it there was some local paths in my machine and because jenkins work as jenkins user i had some issue with that, so i generate a new kubeconfig file with certificates value inside it not as paths with the command : 
`kubectl config view --flatten --minify > kubeconfig-jenkins.yaml`
and i update the credential instead of using the config file i put kubeconfig-jenkins.yaml file and this solve the issue
