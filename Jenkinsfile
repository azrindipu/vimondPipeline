def DEPLOY_APPLICATION = params.getOrDefault("deploy_application", "")
pipeline {
    agent any
    parameters {
        booleanParam(defaultValue: DEPLOY_APPLICATION, description: '', name: 'deploy_application');

    }

    environment {
        ANSIBLE_GIT_URL='https://github.com/azrindipu/vimondAnsible.git'
        PROJECT_GIT_URL='https://github.com/azrindipu/VimondApp.git'
        JAR_LOCATION='${WORKSPACE}/application-code/target'
    }

    stages {
        stage('Getting application code from GIT') {
            steps {
                dir('application-code') {
                    git branch: 'main',
                        url: "${env.PROJECT_GIT_URL}"
                }
            }
        }
        
        stage ('Run unit test') {
            steps {
                dir('application-code') {
                    sh 'mvn test'
                }
            }
        }

        stage ('Compile code') {
            steps {
                dir('application-code') {
                    sh 'mvn -B -DskipTests clean package'
                }
            }
        }

        stage ('Getting ansible code from GIT') {
            steps {
                dir('ansible-code') {
                    git branch: 'main',
                        url: "${env.ANSIBLE_GIT_URL}"
                }
            }
        }

        stage ('Running ansible') {
            steps {
                dir('ansible-code') {
                    ansiblePlaybook credentialsId: 'JenkinsId',
                        disableHostKeyChecking: true,
                        installation: 'ansible',
                        inventory: 'inventory/servers.txt',
                        playbook: 'playbook.yml',
                        extras: "--extra-vars \"deploy_application=${DEPLOY_APPLICATION} package_file_location=${env.JAR_LOCATION}\""
                }
            }
        }
    }

    post {
        always {
            script {
                if(BUILD_NUMBER.toInteger() > 1) {
                    // clean up our workspace
                    echo 'I am DONE !!!'
                    deleteDir()
                }
            }
        }
    }
}
