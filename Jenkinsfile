pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool "sonar-scanner"
    }
    
    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Prateekchhn9/FullStack-Blogging-App.git'
            }
        }
        stage('Compile') {
            steps {
               sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Trivy file scan') {
            steps {
                sh 'trivy fs --format table -o fs.html .'
            }
        }
        stage('SonarQube - code quality check') {
            steps {
                withSonarQubeEnv('sonar-server') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Blogging-app \
                -Dsonar.projectKey=Blogging-app \
                -Dsonar.java.binaries=target'''
                }
            }
        }
        stage('Package build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Deploy to artifectory Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                   sh 'mvn deploy'
                }
            }
        }
        stage('Docker build and tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t prateekchhn9/blogging-app:latest .'
                    }
                }
            }
        }
        stage('Trivy image scan') {
            steps {
                sh 'trivy image --format table -o image.html prateekchhn9/blogging-app:latest'
            }
        }
        stage('Docker Push to dockerhub private registry') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push prateekchhn9/blogging-app:latest'
                    }
                }
            }
        }
        stage('K8 deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: ' prtk-cluster', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://9D618708612B9D4F9E30F58B2790D0AF.gr7.us-east-1.eks.amazonaws.com') {
                sh 'kubectl apply -f deployment-service.yml'
                sleep 10
                }
            }
        }
        stage('k8 verification') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: ' prtk-cluster', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://9D618708612B9D4F9E30F58B2790D0AF.gr7.us-east-1.eks.amazonaws.com') {
                sh 'kubectl get po -o wide'
                sh 'kubectl get svc'
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
                to: 'Prateekchhn9@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
            )
        }
    }
}
}
