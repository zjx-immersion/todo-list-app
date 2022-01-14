pipeline {
    environment {
        registry = "local cache"
        registryCredential = 'dockerhub_id'
        dockerImage = 'todo-list-app:1.0'
}
    agent {
        docker {
            image 'maven:3-alpine'
            args '-v /root/.m2:/root/.m2'
        }
    }
    stages {

        stage('Clone repository') {
            steps {
            /* Clone our repository */
                checkout scm
            }    
        }

        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Deliver') {
            steps {
                sh './jenkins/scripts/deliver.sh'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
    }
}

stage('Build image') {
    node {
        withEnv([
            'SERVICE=todo-list-app',
            'TAG=1.0'
        ]){
        /* Build the docker image */
            sh "echo clear <none docker images>"
            sh "docker rmi -f $(docker images | grep "<none>" | awk "{print \$3}")"
            sh "docker build --no-cache -t ${SERVICE}:${TAG} ."
        }
    }
}

stage('Deploy to k8s') {
    node {
        withEnv([
            'SERVICE=todo-list-app',
            'NAMESPACE=default'
        ]){
            /* Apply all manifest files */
            sh "pwd"
            sh "cat /root/.kube/ca.crt"
            sh "./deployment/deploy.sh"
        }
    }    
}