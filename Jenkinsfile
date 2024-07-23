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
        nexusUrl = '65.0.127.21:8081'
    }
    stages {
        stage("Node Versions Checking") {
            steps {
                script {
                    try {
                        sh '''
                        echo "Checking Node.js version:"
                        node -v
                        echo "Checking npm version:"
                        npm -v
                        '''
                    } catch (Exception e) {
                        echo "Node.js or npm is not available."
                        throw e
                    }
                }
            }
        }
        // stage("git checkout"){
        //     steps{
        //         git branch: 'main', url: 'https://github.com/ullagallu123/expense.git'
        //     }
        // }
        stage("Reading app version") {
            steps {
                script {
                    // Read the package.json file and get the version
                    def packageJson = readJSON file: 'package.json'
                    env.APP_VERSION = packageJson.version
                    echo "Application version: ${env.APP_VERSION}"
                }
            }
        }
        stage("Install the app") {
            steps {
                script {
                    sh """
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
                    zip -q -r ${zipFileName} * -x Jenkinsfile -x ${zipFileName} -x Dockerfile -x backend.pkr.hcl
                    """
                }
            }
        }
        stage("Upload Artifact") {
            steps {
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
