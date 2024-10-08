pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('SCM Checkout') {
            steps {
                git credentialsId: 'git-cred', url: 'https://github.com/AASAITHAMBI573/FullStack-Blogging-App.git'
            }
        }
        
        stage('Maven Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Maven Test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage('Sonarqube Analyisis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Blogging-app -Dsonar.projectKey=Blogging-app -Dsonar.java.binaries=target"
                }
            }
        }
        
        stage('Maven Build') {
            steps {
                sh "mvn package"
            }
        }
        
        stage('Publish To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t aasaithambi5/bloggingapp:latest ."
                    }
                }
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html aasaithambi5/bloggingapp:latest"
            }
        }
        
        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push aasaithambi5/bloggingapp:latest"
                    }
                }
            }
        }
        
        stage('Deploy To Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: ' devopsshack-cluster', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://56E026D010617B50DF4F4E6FE06B58C4.gr7.us-east-2.eks.amazonaws.com') {
                    sh "kubectl apply -f deployment-service.yml -n webapps"
                    sleep 30
                }
            }
        }
        
        stage('Verify Deploment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: ' devopsshack-cluster', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://56E026D010617B50DF4F4E6FE06B58C4.gr7.us-east-2.eks.amazonaws.com') {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
    
    post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'aasaiawsdevops57@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html,trivy-fs-report.html'
            )
        }
    }
}
}

