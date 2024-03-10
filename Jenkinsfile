@Library('my-shared-library') _

pipeline {
    agent any

    parameters {
        choice(name: 'action', choices: ['create', 'delete'], description: 'Choose create/destroy')
        string(name: 'ImageName', description: "Name of the docker build", defaultValue: 'javapp')
        string(name: 'ImageTag', description: "Tag of the docker build", defaultValue: 'v1')
        string(name: 'DockerHubUser', description: "Name of the DockerHub User", defaultValue: 'testingkyaw')
        string(name: 'JFrogURL', description: "JFrog URL", defaultValue: '.')
    }

    stages {
        stage('Git Checkout') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                git branch: "main",
                    credentialsId: 'your-credentials-id',
                    url: "https://github.com/kyawzawaungdevops/Complete-DevOps-CI-CD-pipeline-using-GitOps-with-ArgoCD.git"
            }
        }

        stage('Unit Test maven') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    mvnTest()
                }
            }
        }

        stage('Integration Test maven') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    mvnIntegrationTest()
                }
            }
        }

        stage('Static code analysis: Sonarqube') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    def SonarQubecredentialsId = 'sonarqube-api'
                    staticCodeAnalysis(SonarQubecredentialsId)
                }
            }
        }
        stage('Maven Build : maven') {
            when { 
                expression { params.action == 'create' }
            }
            steps {
                script {
                    mvnBuild()
                }
            }
        }
        stage('Jar file Push : Jfrog ') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: "jfrog",
                        usernameVariable: "USER",
                        passwordVariable: "PASS"
                    )]) {
                        sh "jfrog rt config --url=${params.JFrogURL}/artifactory --user=$USER --password=$PASS --interactive=false"
                        sh "jfrog rt u '/var/lib/jenkins/workspace/Spring Boot Application pipeline/target/spring-petclinic-0.0.1-SNAPSHOT.jar' 'spring-boot-app' --recursive=true"
                    }
                }
            }
        }

        stage('Docker Image Build') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    dockerBuild("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
                }
            }
        }

        stage('Docker Image Scan: trivy ') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    dockerImageScan("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
                }
            }
        }

        stage('Docker Image Push : DockerHub ') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    dockerImagePush("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
                }
            }
        }

        stage('Docker Image Cleanup : DockerHub ') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    dockerImageCleanup("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
                }
            }
        }
    }
}
