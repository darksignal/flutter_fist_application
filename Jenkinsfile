pipeline {
    agent {label 'dev'}

    environment {
        // Define environment variables
        SNYK_OUTPUT_FILE_PATH = 'snyk-results.json'
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
                bat 'flutter --version'
                bat 'flutter doctor'
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
        
        stage('Test Web App') {
            steps {
                // Run Flutter web app tests
                //bat "flutter test test/widget_test.dart"
                //Snyk code scan on the project and save the results to a log file
                /*snykSecurity (
                    failOnError: false, 
                    failOnIssues: false, 
                    organisation: 'darksignal', 
                    projectName: 'darksignal/flutter_first_application', 
                    severity: 'medium', 
                    snykInstallation: 'Snyk@latest', 
                    snykTokenId: 'darksignal-snyk-api-token'
                )*/
                bat 'snyk auth'
                bat 'snyk code test --severity-threshold=high --json-file-output=$SNYK_OUTPUT_FILE_PATH'
                bat 'snyk test --severity-threshold=high --json'
            }
        }
/*
        stage('Deploy Firebase') {
            steps {
                // Run Snyk code scan on the project and save the results to a log file
                bat 'firebase experiments:enable webframeworks'
                //sh 'firebase init hosting'
                bat 'firebase deploy'
            }
        }
*/ 
    }

    post {
        failure {
            // Handle failure (e.g., send notifications)
            slackSend (
                channel: '#jenkins', 
                message: "The CI/CD pipeline for your Flutter web app has failed. Please investigate.\\n\\nPipeline URL: ${currentBuild.absoluteUrl}",
                color: 'danger'
            )
            //Attach file to slack message
            slackUploadFile (
                channel: '#jenkins', 
                filePath: SNYK_OUTPUT_FILE_PATH, 
                initialComment: 'Snyk scan results for the CI/CD pipeline for your Flutter web app.'
            )
        }
        success {
            // Handle success (e.g., send notifications)
            mail (
                bcc: '', 
                body: 'The CI/CD pipeline for your Flutter web app has succeeded. Deployment is complete.\\n\\nPipeline URL: ${currentBuild.absoluteUrl}', 
                cc: '', 
                from: '', 
                replyTo: '', 
                subject: 'CI/CD Pipeline Succeeded: ${currentBuild.fullDisplayName}', 
                to: 'davidromerog@gmail.com'
            )
            slackSend (
                channel: '#jenkins', 
                message: "The CI/CD pipeline for your Flutter web app has succeeded. Deployment is complete.\\n\\nPipeline URL: ${currentBuild.absoluteUrl}",
                color: 'good'
            )
        }
    }
}
