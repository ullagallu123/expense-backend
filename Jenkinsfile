pipeline {
    agent {
        label 'agent'
    }
    options {
        timeout(time: 5, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment {
        PATH = "${HOME}/.nvm/versions/node/v20.15.1/bin:${env.PATH}"
        nexusUrl = "65.0.127.21:8081"
    }
    stages {
        stage("Reading app version") {
            steps {
                script {
                    def packageJson = readJSON file: 'backend/package.json'
                    env.APP_VERSION = packageJson.version
                    echo "Application version: ${env.APP_VERSION}"
                }
            }
        }
        stage("Install the app") {
            steps {
                script {
                    sh """
                    cd backend/
                    npm install
                    ls -ltr
                    echo "Application version: ${env.APP_VERSION}"
                    """
                }
            }
        }
        stage("Build the app") {
            steps {
                script {
                    def zipFileName = "backend-${env.APP_VERSION}-${env.BUILD_NUMBER}.zip"
                    sh """
                    cd backend
                    zip -q -r ${zipFileName} * -x Jenkinsfile -x ${zipFileName} -x Dockerfile -x backend.pkr.hcl
                    mv ${zipFileName} ../
                    ls -ltr ../${zipFileName}
                    """
                }
            }
        }
        stage("Upload Artifact") {
            steps {
                sh """
                ls -ltr
                """
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense',
                        version: "${env.APP_VERSION}",
                        repository: 'backend',
                        credentialsId: 'nexus-auth',
                        artifacts: [
                            [artifactId: 'backend',
                            classifier: '',
                            file: "backend-${env.APP_VERSION}-${env.BUILD_NUMBER}.zip",
                            type: 'zip']
                        ]
                    )
                }
            }
        }
    }
    post {
        always {
            deleteDir()
        }
    }
}
