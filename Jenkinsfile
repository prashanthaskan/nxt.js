pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                // Clone the GitHub repository from the staging branch
                git branch: 'staging', url: 'https://github.com/prashanthaskan/nxt.js'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install Node.js dependencies
                sh 'npm install'
                echo 'Installed dependencies:'
                sh 'npm list --depth=0'
            }
        }

        stage('Backup Existing Files') {
            steps {
                script {
                    // Define the backup directory
                    def backupDir = '/var/www/html/angular-backup'
                    def targetDir = '/var/lib/jenkins/workspace/angular-build' // This should point to the previous workspace

                    // Create backup directory if it doesn't exist
                    if (!fileExists(backupDir)) {
                        sh "mkdir -p ${backupDir}"
                    }

                    // Create a timestamped backup
                    def timestamp = new Date().format("yyyyMMddHHmmss")
                    sh "cp -r ${targetDir} ${backupDir}/backup-${timestamp}"

                    // Limit the number of backups to 7
                    def backups = sh(script: "ls -1 ${backupDir}", returnStdout: true).trim().split("\n")
                    if (backups.size() > 7) {
                        // Sort backups and delete the oldest
                        backups.sort()
                        sh "rm -rf ${backupDir}/${backups[0]}"
                    }
                }
            }
        }

        stage('Build Angular Project') {
            steps {
                // Build the Angular project for production
                sh 'ng build --configuration production'
            }
        }

        stage('Deploy Changed Files') {
            steps {
                script {
                    // Define the target directory
                    def targetDir = '/var/www/html/angular/my-angular'
                    def distDir = 'dist/my-angular' // Define the DIST_DIR variable

                    // Check if the directory exists, create it if it doesn't
                    if (!fileExists(targetDir)) {
                        sh "mkdir -p ${targetDir}"
                    }

                    // Copy only changed files to the target directory
                    sh "cp -ru '${distDir}'/* '${targetDir}/'" // Use quotes to handle spaces

                    // Display the changed files
                    echo 'Changed files:'
                    sh "git diff --name-only HEAD HEAD~1"
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
