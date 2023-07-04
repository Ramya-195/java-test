pipeline {
    agent any

    environment {
        function_name = 'jenkins'
    }
   parameters {
        string(name: 'PARAMETER_NAME', defaultValue: 'default_value', description: 'Parameter description')
        booleanParam(name: 'ENABLE_FEATURE', defaultValue: true, description: 'Enable feature flag')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'test', 'prod'], description: 'Select deployment environment')
    }

    stages {
        stage('Build') {
            steps {
                echo 'Build'
                sh 'mvn package'
            }
        }
        
        stage('SonarQube analysis') {
            when {
                anyOf {
                    branch 'main'
                }
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    echo 'Scanning'
                    sh 'mvn sonar:sonar'
                }
            }
        }
        
        stage("Quality Gate") {
            steps {
                script {
                    try {
                        timeout(time: 1, unit: 'MINUTES') {
                            def qualityGate = waitForQualityGate abortPipeline: true
                            echo "Quality Gate status is ${qualityGate.status}"
                            echo "Quality Gate details: ${qualityGate}"
                        }
                    } catch (Exception e) {
                        echo "Quality Gate failed: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Push') {
            steps {
                echo 'Push'
                sh "aws s3 cp target/sample-1.0.3.jar s3://ramyas3bucket1"
            }
        }

        stage('Deploy to Test') {
            steps {
                echo 'Build'
                sh "aws lambda update-function-code --function-name $function_Test --region us-east-1 --s3-bucket ramyas3bucket1 --s3-key sample-1.0.3.jar"
            }
        }

        stage('Deploy to Prod') {
            steps {
                when {
                    expression { params.ENVIRONMENT == 'Prod' }
                }
                echo 'Build'
                input(message: 'Are we good for production?')
                sh "aws lambda update-function-code --function-name $function_Prod --region us-east-1 --s3-bucket ramyas3bucket1 --s3-key sample-1.0.3.jar"
            }
        }
    }

    post {
        always {
            mail(
                body: 'Whatever',
                subject: 'Jenkins Build Notification',
                to: 'ramyairganti@gmail.com'
            )
        }
    }
}
