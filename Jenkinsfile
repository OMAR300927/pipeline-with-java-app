pipeline {
    agent any
    
    tools {
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/OMAR300927/pipeline-with-java-app'
            }
        }
        
        stage('mvn clean') {
            steps {
                sh 'mvn clean'
            }
        }
        
        stage('mvn compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('mvn test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner \
                            -Dsonar.projectName=JavaApp \
                            -Dsonar.projectKey=JavaApp \
                            -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('Quality gate') {
            steps {
                waitForQualityGate abortPipeline: true, credentialsId: 'sonar-cred'
            }
        }
        
        stage('Scan file system') {
            steps {
                sh 'trivy fs . --format table -o trivy-fs-report.html'
            }
        }
        
        stage('mvn package') {
            steps {
                sh 'mvn package'
            }
        }
        
        stage('Build & tag image') {
            steps {
                sh 'docker build -t omarsa999/javaimage:latest .'
            }
        }
        
        stage('Scan the image') {
            steps {
                sh 'trivy image --format table -o trivy-image-report.html omarsa999/javaimage:latest'
            }
        }
        
        stage('Push the image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker push omarsa999/javaimage:latest'
                }
            }
        }
        
        stage('Apply kubernetes cluster') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) { 
                    sh '''
                    kubectl config get-contexts
                    kubectl apply -f ./K8s/ '''
                }
            }
        }
    }
    
    post {
        success {
            emailext(
                to: 'omsa3008@gmail.com',
                subject: "Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <h3>Build Succeeded</h3>
                    <p>Job: ${env.JOB_NAME}</p>
                    <p>Build Number: ${env.BUILD_NUMBER}</p>
                    <p>Check details: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """,
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
            )
        }
        failure {
            emailext(
                to: 'omsa3008@gmail.com',
                subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <h3>Build Failed</h3>
                    <p>Job: ${env.JOB_NAME}</p>
                    <p>Build Number: ${env.BUILD_NUMBER}</p>
                    <p>Check details: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """,
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
            )
        }
    }
}
