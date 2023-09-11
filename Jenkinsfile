pipeline {
    agent {label 'dev'}

    environment {
        // Define environment variables
        //FLUTTER_HOME = '/usr/local/flutter'
        DOCKER_IMAGE_NAME = 'davidromerog/flutter_fist_application'
        DOCKER_REGISTRY_CREDENTIALS = credentials('docker-registry-credentials')
        SNYK_LOG_FILE = 'snyk-scan.log' // Define the Snyk log file
        SERVER_IP = '192.168.68.54'
        SSH_USER = 'drg'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from GitHub
                git credentialsId: 'Github-Token-Classic', url: 'https://github.com/darksignal/flutter_fist_application.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install Flutter dependencies
                //sh '''
                //    curl -o flutter.tar.xz https://storage.googleapis.com/flutter_infra/releases/stable/linux/flutter_linux_2.0.6-stable.tar.xz
                //    tar xf flutter.tar.xz -C /usr/local
                //    export PATH="$PATH:$FLUTTER_HOME/bin"
                //    flutter --version
                //    flutter pub get
                //'''
                bat 'flutter --version'
                bat 'flutter doctor'
                //sh 'sudo snap install flutter --classic'
                bat 'flutter pub get'
                bat 'flutter clean'
            }
        }

        stage('Build Web App') {
            steps {
                // Build Flutter web app
                bat "flutter build web --web-renderer canvaskit --release"
            }
        }
/*
        stage('Snyk Code Scan') {
            steps {
                // Run Snyk code scan on the project and save the results to a log file
                //sh "snyk test --all-projects > $SNYK_LOG_FILE"
                //snykSecurity(
                //    snykInstallation: 'Snyk@latest', // Specify the Snyk installation to use
                //    snykTokenId: 'darksignal-snyk-api-token', // Specify the Snyk API token to use 
                //    targetFile: $SNYK_LOG_FILE, // Specify the target file to use
                //    additionalArguments: '--all-projects' // --file=$SNYK_LOG_FILE" // Specify the log file to use
                snykSecurity failOnIssues: false, projectName: 'flutter_fist_application', severity: 'medium', snykInstallation: 'Snyk@latest', snykTokenId: 'darksignal-snyk-api-token', targetFile: SNYK_LOG_FILE
                )
            }
        }
*/
        stage('Deploy Firebase') {
            steps {
                // Run Snyk code scan on the project and save the results to a log file
                bat 'firebase experiments:enable webframeworks'
                //sh 'firebase init hosting'
                bat 'firebase deploy'
            }
        }
/*
        stage('Build Docker Image') {
            steps {
                // Build Docker image
                script {
                    docker.build(DOCKER_IMAGE_NAME, './build/web')
                }
            }
        }

        stage('Snyk Container Scan') {
            steps {
                // Run Snyk container scan on the Docker image and append the results to the log file
                sh "snyk container test ${DOCKER_IMAGE_NAME} >> $SNYK_LOG_FILE"
            }
        }

        stage('Push Docker Image') {
            steps {
                // Push Docker image to Docker registry
                script {
                    docker.withRegistry('', DOCKER_REGISTRY_CREDENTIALS) {
                        docker.image(DOCKER_IMAGE_NAME).push()
                    }
                }
            }
        }

        stage('Deploy???') {
            steps {
                // Deploy your Docker container to your chosen environment
                // You may need to use a tool like Docker Compose or Kubernetes here
                // For example, using Docker Compose:
                sh 'docker-compose up -d'
            }
        }

        stage('DeployRPi') {
            steps {
                script {
                    def remoteDir = "~/Downloads" // Replace with your remote directory path
                    def dockerRunCmd = "docker run -d -p 80:80 $DOCKER_IMAGE_NAME"
                    
                    sshagent(credentials: ['linux-ssh']) {
                        // Copy the Docker image to the remote server
                        sshPut(
                            remote: "${remoteDir}/your-flutter-web-app.tar",
                            from: "${DOCKER_IMAGE_NAME}.tar"
                        )

                        // SSH into the remote server and execute Docker run command
                        sshCommand(
                            remote: remoteDir,
                            user: SSH_USER,
                            password: credentials('linux-ssh'),
                            command: dockerRunCmd,
                            failonerror: true
                        )
                    }
                }
            }
        }
 */   
    }

    post {
        failure {
            // Handle failure (e.g., send notifications)
            emailext(
                subject: "CI/CD Pipeline Failed: ${currentBuild.fullDisplayName}",
                body: "The CI/CD pipeline for your Flutter web app has failed. Please investigate.\n\nPipeline URL: ${currentBuild.absoluteUrl}",
                to: 'davidromerog@gmail.com',
                replyTo: 'hola@grantigris.com',
                attachLog: true, // Attach the Snyk log file to the email
                attachmentsPattern: '$SNYK_LOG_FILE' // Specify the log file to attach
            )
        }
        success {
            // Handle success (e.g., send notifications)
            emailext(
                subject: "CI/CD Pipeline Succeeded: ${currentBuild.fullDisplayName}",
                body: "The CI/CD pipeline for your Flutter web app has succeeded. Deployment is complete.\n\nPipeline URL: ${currentBuild.absoluteUrl}",
                to: 'davidromerog@gmail.com',
                replyTo: 'hola@grantigris.com'
            )
        }
    }
}
