pipeline {
    agent any
    
    tools {
        maven 'maven3'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/praveenselvakumarasamy/Todo_Task_App.git'
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar_server') {
                    sh '''mvn sonar:sonar -Dsonar.projectName=todo_task_app \
                    -Dsonar.projectKey=todo_task_app  -Dsonar.java.binaries=target'''
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o report.html .'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage('Publish On Nexus') {
            steps { 
                withMaven(globalMavenSettingsConfig: 'Nexus_setting', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }
        
        stage('Build and Push Docker Image') {
            environment{
                DOCKER_IMAGE = "praveenselvakumarasamy/todo_task_app:latest"
            }
            
            steps {
                script{
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                    withDockerRegistry(credentialsId: 'docker_cred') {
                        sh 'docker push ${DOCKER_IMAGE}'
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --format table -o report1.html praveenselvakumarasamy/todo_task_app:latest'
            }
        }
    }
}
