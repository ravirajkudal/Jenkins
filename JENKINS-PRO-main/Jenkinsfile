pipeline {
    agent any
    
    // Tools required for the build
    tools {
        nodejs 'Node18' // Matches the name configured in Manage Jenkins -> Tools
    }

    // Parameters to trigger manual builds with options
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'staging', 'prod'], description: 'Select deployment environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Uncheck to skip tests')
    }

    // Global Environment variables
    environment {
        APP_NAME = "MySampleApp"
        // Dummy credentials mapping
        DEPLOY_TOKEN = credentials('dummy-deploy-token') 
    }

    stages {
        stage('Build') {
            steps {
                echo "Building ${APP_NAME} for ${params.DEPLOY_ENV} environment..."
                sh 'npm install'
            }
        }

        stage('Test') {
            // 'when' condition based on the parameter
            when {
                expression { params.RUN_TESTS == true }
            }
            // Parallel execution to speed up pipelines
            parallel {
                stage('Unit Tests') {
                    steps {
                        echo "Running Unit Tests..."
                        sh 'npm run test:unit'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        echo "Running Integration Tests..."
                        sh 'npm run test:integration'
                    }
                }
            }
        }

        stage('Deploy') {
            // 'when' condition based on branch and parameter
            when {
                branch 'main'
                environment name: 'DEPLOY_ENV', value: 'prod'
            }
            steps {
                echo "Deploying to Production securely..."
                // Masking credentials in logs while using them
                sh '''
                    echo "Connecting to server using token: ${DEPLOY_TOKEN}"
                    echo "Starting node server..."
                    node app.js > app.log 2>&1 &
                '''
            }
        }
    }

    // Post-build actions based on pipeline status
    post {
        always {
            echo "Pipeline execution finished. Cleaning up workspace..."
            cleanWs() // Cleans the workspace
        }
        success {
            echo "SUCCESS: The pipeline ran flawlessly!"
        }
        failure {
            echo "FAILURE: Something went wrong. Check the logs."
        }
    }
}

