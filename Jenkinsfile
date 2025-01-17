def image
def tag
pipeline {
    agent { 
        label 'master'
        }
    triggers { pollSCM('* * * * *') }
    options {
        timestamps()
    }
    stages {
        stage("pylint revision") {
            steps {
                sh 'pylint  --disable=missing-docstring  application/diploma_app.py'
            }
        }    
        stage("docker build") {
            steps {
                script {
                    dir("${pwd()}/application") {                       
                        def sha = env.GIT_COMMIT.substring(0,8)
                        tag = "${env.GIT_BRANCH}-${sha}"
                        image = docker.build( "alexv77745/diploma:${tag}")
                    }
                }
            }
        }    
        stage("docker push") {
            steps {
                script {  
                    docker.withRegistry('https://registry.hub.docker.com','docker_hub_alexv77745') {                    
                        image.push()
                        image.push('latest')
                    }
                }
            }
        } 
        stage("trigger deploy") {
            steps {
                    build job: 'Deploy_Job', parameters: [
                        [$class: 'StringParameterValue', name: 'stackName', value: 'myapp-dev'],
                        [$class: 'StringParameterValue', name: 'DOCKER_IMAGE', value: tag]
                    ]
            }    
        }      
    }
}    
