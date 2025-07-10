pipeline {
    agent any

    triggers {
        githubPush()
    }

    options {
        timeout(time: 30, unit: 'MINUTES') // Timeout for the entire pipeline
        buildDiscarder(logRotator(numToKeepStr: '7')) // Discard old builds to save disk space
        disableConcurrentBuilds() // Ensures that only one build can run at a time
        timestamps() // Adds timestamps to the console output
        skipDefaultCheckout() // Skips the default checkout of source code, useful if you're doing a custom checkout
        retry(3) // Automatically retries the entire pipeline up to 3 times if it fails
    }

    environment {
        DOCKER_HUB_USERNAME = "tchuinsu"
        ALPHA_APPLICATION_01_REPO = "alpha-application-011"
        ALPHA_APPLICATION_02_REPO = "alpha-application-022"
        DOCKER_CREDENTIAL_ID = 'docker-hub-creds'
    }

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 's3coubis-01', description: 'Git branch to build')
        string(name: 'APP1_TAG', defaultValue: 'latest', description: 'Tag for Application 011')
        string(name: 'APP2_TAG', defaultValue: 'latest', description: 'Tag for Application 022')
        string(name: 'PORT_APP11', defaultValue: '8085', description: 'Host port for App 011')
        string(name: 'PORT_APP22', defaultValue: '8086', description: 'Host port for App 022')
        string(name: 'PORT_ON_DOCKER_HOST', defaultValue: '', description: 'Port on Docker host')
    }

    stages {
        //stage ('Check Allow Users') {
        //    steps {
        //        script {
        //            wrap([$class: 'BuildUser']) {
        //                def build_id = env.BUILD_USER_ID
        //                def build_user = env.BUILD_USER
        //                echo "build_id : $build_id"
        //                if (build_id in ['tchuinsu', 'admin']) {
        //                    echo "Hi $build_user, You are allowed to run this job"
        //                } else {
        //                    error "Hi $build_user, You are not allowed to run this job"
        //                }
        //            }
        //        }
        //    }
        //}
        stage('Sanity Check') {
            steps {
                script {
                    sanity_check()
                }
            }
        }

        stage('Clone Repository') {
            when {
                expression {
                    params.BRANCH_NAME == 's3coubis-01'
                }
            }
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${params.BRANCH_NAME}"]],
                        userRemoteConfigs: [[
                            credentialsId: 'github-auth',
                            url: 'git@github.com:tchuinsu/s8-web-2-Tia.git'
                        ]]
                    ])
                }
            }
    
        stage('Check Code Structure') {
            steps {
                sh 'ls -l'
                sh 'pwd'
            }
        }

        stage('Build Application 011') {
            steps {
                sh """
                    docker build -t ${env.DOCKER_HUB_USERNAME}/${env.ALPHA_APPLICATION_01_REPO}:${params.APP1_TAG} -f application-011.Dockerfile .
                    docker images
                """
            }
        }

        stage('Build Application 022') {
            steps {
                sh """
                    docker build -t ${env.DOCKER_HUB_USERNAME}/${env.ALPHA_APPLICATION_02_REPO}:${params.APP2_TAG} -f application-022.Dockerfile .
                    docker images
                """
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: env.DOCKER_CREDENTIAL_ID,
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                }
            }
        }

        stage('Push Application 011 to DockerHub') {
            steps {
                sh "docker push ${env.DOCKER_HUB_USERNAME}/${env.ALPHA_APPLICATION_01_REPO}:${params.APP1_TAG}"
            }
        }

        stage('Push Application 022 to DockerHub') {
            steps {
                sh "docker push ${env.DOCKER_HUB_USERNAME}/${env.ALPHA_APPLICATION_02_REPO}:${params.APP2_TAG}"
            }
        }

        stage('Deploy Application 011') {
            steps {
                script {
                    sh """
                        # Stop and remove container if it exists
                        docker rm -f app011 || true
        
                        # Run new container
                        docker run -itd --name app011 -p ${params.PORT_APP11}:80 ${env.DOCKER_HUB_USERNAME}/${env.ALPHA_APPLICATION_01_REPO}:${params.APP1_TAG}

                        docker ps
                    """
                }
            }
        }



        stage('Deploy Application 022') {
            steps {
                script {
                    sh """
                        # Stop and remove container if it exists
                        docker rm -f app022 || true

                        # Run new container
                        docker run -itd --name app022 -p ${params.PORT_APP22}:80 ${env.DOCKER_HUB_USERNAME}/${env.ALPHA_APPLICATION_02_REPO}:${params.APP2_TAG}

                        docker ps
                    """
                }
            }
        }
    }
}

def customFunction() {
    sh """
        ls -l
        pwd
        uptime
    """
}


def sanity_check() {
    if (params.BRANCH_NAME.isEmpty()){
       echo "The parameter BRANCH_NAME is not set"
       sh 'exit 2'
   } 
}