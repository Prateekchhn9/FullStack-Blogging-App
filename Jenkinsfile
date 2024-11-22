pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool "sonar-scanner"
    }
    
    stages {
        stage('git checkout') {
            steps {
                echo 'Git Checkout'
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Prateekchhn9/FullStack-Blogging-App.git'
            }
        }
        stage('Compile') {
            steps {
                echo 'Compile'
                sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
                echo 'test'
                sh "mvn test"
            }
        }
        stage('Trivy FS scan') {
            steps {
                echo 'Trivy FS scan'
                sh "trivy fs --format table -o fs.html ."
            }
        }
        stage('Soanrqube Analysis') {
            steps {
                echo 'Soanrqube Analysis'
                withSonarQubeEnv('sonar-server') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Blogging-app -Dsonar.projectKey=Blogging-app \
                        -Dsonar.java.binaries=target'''
                }
                
            }
        }
        stage('Build') {
            steps {
                echo 'Build step'
                sh "mvn package"
            }
        }
        stage('Publish Artifects') {
            steps {
                echo 'Publish Artifects'
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
    }
}
