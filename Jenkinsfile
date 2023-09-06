pipeline {
    agent jenkins-slave
    tools {
        maven 'mymaven'
        docker 'mydocker'
    }

    environment {
        registry = "greyabiwon/readyvpro"
        registryCredential = 'docker-login'
        aws-cred = '578d39b2-d05e-49aa-8aa5-7ed15caccd50'
        sonar-token = 'sonar-token'
        s3Bucket = 'alb-bucket-grey'  // Your S3 bucket name
        artifactName = 'vprofile-v2.war'  // Name of the artifact
        SLACK_TOKEN = 'slack-token'
    }
}
    stages {
        stage('Checkout') {
            steps {
                // Checkout your source code from your version control system
                checkout scm
            }
        }

    stages {
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

        stage('Run SonarCloud Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: '', installationName: 'sonar-server') {
                        // Run SonarCloud analysis
                        sh "mvn sonar:sonar -Dsonar.login=${sonar-token}"
                    }
                }
            }
        }

        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    // Define the Docker image name and tag (replace with your actual image name and tag)
                    def dockerImageName = "${registry}:${BUILD_NUMBER}"

                    // Run Trivy scan on your Docker image
                    def trivyScanResult = sh(script: "trivy --severity HIGH,CRITICAL ${dockerImageName}", returnStatus: true)

                    if (trivyScanResult == 0) {
                        echo 'Trivy scan passed. No vulnerabilities found.'
                    } else {
                        error 'Trivy scan failed. Vulnerabilities detected.'
                    }
                }
            }
}

        stage('Deploy Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push("$BUILD_NUMBER")
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Remove Unused docker image') {
            steps {
                sh "docker rmi $registry:$BUILD_NUMBER"
            }
        }

        stage('Upload Artifact to S3') {
            steps {
                script {
                    def awsCli = bat(script: 'aws', returnStatus: true)
                    if (awsCli == 0) {
                        // AWS CLI is installed and available
                        def objectKey = artifactName  // Object key is the artifact name
                        def artifactPath = "target/${artifactName}"  // Path to your generated artifact

                        // Use the AWS CLI to upload the artifact to S3
                        def awsUploadCmd = "aws s3 cp ${artifactPath} s3://${s3Bucket}/${objectKey} --region your-aws-region"

                        sh(awsUploadCmd)
                    } else {
                        error 'AWS CLI is not installed. Please install it or configure AWS credentials.'
                    }
                }
            }
            post {
                failure {
                    slackSend(
                        color: '#FF0000',
                        message: "Pipeline failed: ${currentBuild.fullDisplayName}",
                        tokenCredentialId: 'slack-token',
                        channel: '#devops-cicd'
                    )
                }
                success {
                    slackSend(
                        color: 'good',
                        message: "Pipeline succeeded: ${currentBuild.fullDisplayName}",
                        tokenCredentialId: SLACK_TOKEN,
                        channel: '#devops-cicd'
                    )
                }
            }
        }

        // ... (other stages)
    }
}
