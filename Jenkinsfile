pipeline {
    agent {
        label 'docker'
    }

    environment {
        // Proxy
        http_proxy = 'http://proxy.psnmc.qld.gov.au:3128'
        https_proxy = 'http://proxy.psnmc.qld.gov.au:3128'
        GOPATH="${WORKSPACE}"
        PATH="${GOPATH}/bin:$PATH"
    }

    stages {
        stage('Checkout') {
            steps {
                dir("${GOPATH}/src") {
                    script {
                        if (env.BRANCH_NAME == 'dev' || env.BRANCH_NAME == 'master') {
                            sh """curl -X PUT -d '"building"' https://psnmc-jenkins.firebaseio.com/ssh-bastion-${BRANCH_NAME}.json?auth=6CrNwVrQlzgpdhysYrwRXEZ5WsJQXZy046qYpNoM"""
                        }
                    }
                    checkout scm
                }
            }
        }

        stage('Pre Build') {
            steps {
                dir("${GOPATH}/src") {
                    sh 'echo Workspace: $WORKSPACE'
                    sh 'echo GOPATH: $GOPATH'
                    sh 'go version'
                    sh 'ls'
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Compiling go...'
                echo '========================================='
                sh """cd $GOPATH/src/ssh-bastion && go build -ldflags '-s'"""
            }
        }

        stage('Save Output') {
            steps {
                archiveArtifacts artifacts: "$GOPATH/src/ssh-bastion/ssh-bastion"
            }
        }
    }

    post {
        // failure {
        //     script {
        //         if (env.BRANCH_NAME == 'dev' || env.BRANCH_NAME == 'master') {
        //             slackSend ":poop: Build failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        //             sh """curl -X PUT -d '"failure"' https://psnmc-jenkins.firebaseio.com/ssh-bastion-${BRANCH_NAME}.json?auth=6CrNwVrQlzgpdhysYrwRXEZ5WsJQXZy046qYpNoM"""
        //         }
        //     }
        // }

        success {
            script {
                if (env.BRANCH_NAME == 'dev' || env.BRANCH_NAME == 'master') {
                    sh """curl -X PUT -d '"success"' https://psnmc-jenkins.firebaseio.com/ssh-bastion-${BRANCH_NAME}.json?auth=6CrNwVrQlzgpdhysYrwRXEZ5WsJQXZy046qYpNoM"""
                }
            }
        }

        always {
            cleanWs()
        }
    }
}
