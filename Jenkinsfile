pipeline {
    agent any

    tools {
        // Define your tools here
        maven 'mymaven'
    }

    environment {
        registry = "greyabiwon/readyvpro"
        registryCredential = 'docker-login'
        SONAR_TOKEN = 'SONAR_TOKEN'
        SLACK_TOKEN = 'slack-token'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout your source code from your version control system
                checkout scm
            }
        }

        stage('BUILD') {
            steps {
                sh 'mvn clean install'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST') {
            steps {
                sh 'mvn test'
            }
        }

        stage('CODE ANALYSIS WITH CHECKSTYLE') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=grey-vpro-project -Dsonar.projectName='grey-vpro-project'"
                }
            }
        }
    }
}

        // stage('Trivy Scan') {
        //     steps {
        //         script {
        //             // Define the Docker image name and tag (replace with your actual image name and tag)
        //             def dockerImageName = "${registry}:${BUILD_NUMBER}"

        //             // Run Trivy scan on your Docker image
        //             def trivyScanResult = sh(script: "trivy image ${dockerImageName}", returnStatus: true)

        //             if (trivyScanResult == 0) {
        //                 echo 'Trivy scan passed. No vulnerabilities found.'
        //             } else {
        //                 error 'Trivy scan failed. Vulnerabilities detected.'
        //             }
        //         }
        //     }
        // }

        // Other stages can be added here

        // post {
        //     failure {
        //         slackSend(
        //             color: '#FF0000',
        //             message: "Pipeline failed: ${currentBuild.fullDisplayName}",
        //             tokenCredentialId: 'slack-token',
        //             channel: '#devops-cicd'
        //         )
        //     }
        //     success {
        //         slackSend(
        //             color: 'good',
        //             message: "Pipeline succeeded: ${currentBuild.fullDisplayName}",
        //             tokenCredentialId: SLACK_TOKEN,
        //             channel: '#devops-cicd'
        //         )
        //     }
        
    
