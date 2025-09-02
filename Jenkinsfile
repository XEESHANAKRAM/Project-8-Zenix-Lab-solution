pipeline {
    agent any

    stages {
        stage('github') {
            steps {
                git branch: 'main', url: 'https://github.com/XEESHANAKRAM/Project-8-Zenix-Lab-solution.git'
            }
        }
        stage('Copy files to Ansible host') {
    steps {
        sshagent(['ansible-demo']) {
            sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@172.31.18.66 "rm -rf /home/ubuntu/zenix && mkdir -p /home/ubuntu/zenix"
                scp -o StrictHostKeyChecking=no -r * ubuntu@172.31.18.66:/home/ubuntu/zenix/
            '''
        }
    }
}
       stage('docker image build ') {
    steps {
        sshagent(['ansible-demo']) {
            sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@172.31.18.66 "
                cd /home/ubuntu/zenix && sudo docker build -t zenix:v1.$BUILD_ID ."
            '''
        }
    }
}

        stage('image tagging') {
            steps {
                sshagent(['ansible-demo']) {
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.18.66 "cd /home/ubuntu && sudo docker tag $JOB_NAME:v1.$BUILD_ID xeeshanakram/$JOB_NAME:v1.$BUILD_ID"'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.18.66 "cd /home/ubuntu && sudo docker tag $JOB_NAME:v1.$BUILD_ID xeeshanakram/$JOB_NAME:latest"'
                }
            }
        }
        stage('Docker Push to docker hub') {
    steps {
        sshagent (['ansible-demo']) {
            withCredentials([usernamePassword(credentialsId: 'dockerhub-demo', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                sh """
                ssh -o StrictHostKeyChecking=no ubuntu@172.31.18.66 "
                echo $DOCKER_PASS | sudo docker login -u $DOCKER_USER --password-stdin &&
                sudo docker push xeeshanakram/zenix:v1.$BUILD_ID &&
                sudo docker push xeeshanakram/zenix:latest
                "
                """
            }
        }
    }
    stage('Deploy to Kubernetes via Ansible') {
    sshagent(['k8s_server']) {
        sh '''
            # Create target directory on K8s server
            ssh -o StrictHostKeyChecking=no ubuntu@172.31.30.55 "rm -rf /home/ubuntu/zenix && mkdir -p /home/ubuntu/zenix"
            
            # Copy all repo root files (since everything needed is in root)
            scp -o StrictHostKeyChecking=no * ubuntu@172.31.30.55:/home/ubuntu/zenix/
        '''
    }
}
}
    }
}
