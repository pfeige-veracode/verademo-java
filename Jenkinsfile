// Jenkinsfile for VeraDemo Java with Veracode Scans

pipeline {
    agent { label 'built-in' }

    environment {
        VERACODE_APP_NAME = 'Verademo' // App Name in Veracode Platform
    }

    stages {
        stage('Environment Verify') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'pwd'
                        sh 'ls -la'
                        sh 'echo $PATH'
                    } else {
                        bat 'dir'
                        bat 'echo %PATH%'
                    }
                }
            }
        }

        stage('Build') {
            steps {
                withMaven(maven: 'maven-3') {
                    script {
                        if (isUnix()) {
                            sh 'mvn -f app clean package'
                        } else {
                            bat 'mvn -f app clean package'
                        }
                    }
                }
            }
        }

        stage('Veracode Static Scan') {
            steps {
                echo 'Starting Veracode Static Scan'
                withCredentials([usernamePassword(
                    credentialsId: 'veracode_login',
                    usernameVariable: 'VERACODE_API_ID',
                    passwordVariable: 'VERACODE_API_KEY'
                )]) {
                    script {
                        veracode(
                            applicationName: "${VERACODE_APP_NAME}",
                            criticality: 'VeryHigh',
                            debug: true,
                            scanName: "Jenkins-${BUILD_NUMBER}",
                            uploadIncludesPattern: 'app/target/verademo.war',
                            vid: "${VERACODE_API_ID}",
                            vkey: "${VERACODE_API_KEY}"
                        )
                    }
                }
            }
        }

        stage('Veracode SCA') {
            when { expression { isUnix() } } // Only run SCA on Unix x86
            steps {
                echo 'Starting Veracode SCA Scan'
                withCredentials([string(credentialsId: 'SCA_Token', variable: 'SRCCLR_API_TOKEN')]) {
                    sh "curl -sSL https://download.sourceclear.com/ci.sh | sh -s -- scan app"
                }
            }
        }

        stage('Veracode Container Scan') {
            when { expression { isUnix() } } // Only run container scan on Unix x86
            steps {
                echo 'Starting Veracode Container Scan'
                withCredentials([usernamePassword(
                    credentialsId: 'veracode_login',
                    usernameVariable: 'VERACODE_API_KEY_ID',
                    passwordVariable: 'VERACODE_API_KEY_SECRET'
                )]) {
                    sh '''
                        curl -fsS https://tools.veracode.com/veracode-cli/install | sh
                        ./veracode scan --type directory --source . --format table
                    '''
                }
            }
        }
    }
}
