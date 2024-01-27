pipeline {
    agent {
        label ''
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
        timeout (time: 15, unit: 'MINUTES')
        timestamps()
    }
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: '')
        string(name: 'SONAR_VERSION', defaultValue: '5.0.1.3006', description: '')
    }
    stages {
        stage ('Checkout') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app") {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${env.BRANCH_NAME}"]],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [[$class: 'LocalBranch']],
                        submoduleCfg: [],
                        userRemoteConfigs: [[
                            url: 'https://github.com/cyprientemateu/jenkins-test.git',
                            credentialsId: 'github-auth'
                        ]]
                    ])
                }
            }
        }
        stage('Open sonar-project.properties') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code") {
                    script {
                        // Use 'cat' command to display the content of sonar-project.properties
                        sh 'cat sonar-project.properties'
                    }
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code") {
                    script {
                        withSonarQubeEnv('SonarScanner') {
                            sh "sonar-scanner"
                        }
                    }
                }
            }
        }
        stage('Build auth') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code") {
                    script {
                        // Build the Docker image for auth
                        sh 'sudo docker build -t cyprientemateu/sixfure-auth -f auth/Dockerfile .'
                        sh 'sudo docker images'
                    }
                }
            }
        }
        stage('push auth') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code") {
                    script {
                        // Build the Docker image for auth
                        sh 'sudo docker push cyprientemateu/sixfure-auth:latest'
                    }
                }
            }
        }
        stage('Build db') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code") {
                    script {
                        // Build the Docker image for db
                        sh 'sudo docker build -t cyprientemateu/sixfure-db -f db/Dockerfile .'
                        sh 'sudo docker images'
                    }
                }
            }
        }
        stage('push db') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code") {
                    script {
                        // Build the Docker image for auth
                        sh 'sudo docker push cyprientemateu/sixfure-db:latest'
                    }
                }
            }
        }
        stage('Build redis') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code") {
                    script {
                        // Build the Docker image for redis
                        sh 'sudo docker build -t cyprientemateu/sixfure-redis -f redis/Dockerfile .'
                        sh 'sudo docker images'
                    }
                }
            }
        }
        stage('push redis') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code") {
                    script {
                        // Build the Docker image for auth
                        sh 'sudo docker push cyprientemateu/sixfure-redis:latest'
                    }
                }
            }
        }
        stage('Build ui') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code") {
                    script {
                        // Build the Docker image for ui
                        sh 'sudo docker build -t cyprientemateu/sixfure-ui -f ui/Dockerfile .'
                        sh 'sudo docker images'
                    }
                }
            }
        }
        stage('Build weather') {
            steps {
                dir("${WORKSPACE}/tcc-weather-app/code") {
                    script {
                        // Build the docker image for weather
                        sh 'sudo docker build -t cyprientemateu/sixfure-weather -f weather/Dockerfile .'
                        sh 'sudo docker images'
                    }
                }
            }
        }

        // New stage: Deploy
        // stage('Deploy') {
        //     steps {
        //         dir("${WORKSPACE}/tcc-weather-app") {
        //             script {
        //                 // Deploy the Docker image to a registry or run it on a server
        //                 sh 'docker push your-docker-image-name:latest'

        //                 // Example: Deploy to a Docker Swarm
        //                 sh 'docker stack deploy -c docker-compose.yml your-stack-name'
        //             }
        //         }
        //     }
        // }

    }
    post {
        
        success {
            slackSend color: '#2EB67D',
            channel: 'tcc-lab',  
            message: "*tcc-weather-app Project Build Status*" +
            "\n Project Name: tcc-weather-app" +
            "\n Job Name: ${env.JOB_NAME}" +
            "\n Build number: ${currentBuild.displayName}" +
            "\n Build Status : *SUCCESS*" +
            "\n Build url : ${env.BUILD_URL}"
        }
        failure {
            slackSend color: '#E01E5A', 
            channel: 'tcc-lab',  
            message: "*tcc-weather-app Project Build Status*" +
            "\n Project Name: tcc-weather-app" +
            "\n Job Name: ${env.JOB_NAME}" +
            "\n Build number: ${currentBuild.displayName}" +
            "\n Build Status : *FAILED*" +
            "\n Action : Please check the console output to fix this job IMMEDIATELY" +
            "\n Build url : ${env.BUILD_URL}"
        }
        unstable {
            slackSend color: '#ECB22E',
            channel: 'tcc-lab', 
            message: "*tcc-weather-app Project Build Status*" +
            "\n Project Name: tcc-weather-app" +
            "\n Job Name: ${env.JOB_NAME}" +
            "\n Build number: ${currentBuild.displayName}" +
            "\n Build Status : *UNSTABLE*" +
            "\n Action : Please check the console output to fix this job IMMEDIATELY" +
            "\n Build url : ${env.BUILD_URL}"
        }   
    }
}
