pipeline {
    agent any

    tools {
        // This matches the name you gave the NodeJS tool in Jenkins settings
        nodejs 'node-25'
    }

    environment {
        // The project requires these to run, though for CI we use dummy values
        DATABASE_URL = "file:./dev.db"
        JWT_SECRET   = "jenkins-test-secret"
        NODE_ENV     = "test"
    }

    stages {
        stage('Cleanup') {
            steps {
                echo 'Cleaning up workspace...'
                deleteDir()
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing npm dependencies...'
                sh 'npm ci'
            }
        }

        stage('Lint & Code Quality') {
            steps {
                echo 'Running Lint...'
                // The RealWorld app uses Nx; this checks code style
                sh 'npx nx lint api'
            }
        }

        stage('Database Setup') {
            steps {
                echo 'Generating Prisma Client...'
                sh 'npx prisma generate'
            }
        }

        stage('Test') {
            steps {
                echo 'Running Unit & Integration Tests...'
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the production application...'
                sh 'npx nx build api'
            }
        }
    }

    post {
        always {
            echo 'Archiving build artifacts...'
            // This saves the built files so you can download them from the Jenkins UI
            archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
        }
        failure {
            echo 'Pipeline failed. Check the logs above for errors.'
        }
    }
}
