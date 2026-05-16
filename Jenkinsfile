pipeline{
    agent {
    label 'slave2'
    }
    environment {
        USER_NAME = "phani"
        IP_ADDRESS = "34.142.188.216"
        IMAGE_NAME = "i27-helpdesk-ui:dev"
        CONTAINER_NAME = "i27-ui"
    }
    stages{
        stage('connecting to vm') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no ${USER_NAME}@${IP_ADDRESS} 'echo "connected to test-vm"'
                """
            }
        }
        stage('cloning'){
            steps{
                sh """
                    ssh -o StrictHostKeyChecking=no ${USER_NAME}@${IP_ADDRESS} '
                        git clone https://github.com/i27academy/i27-helpdesk-ui.git
                    '
                """
            }
        }
        stage('image build') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no ${USER_NAME}@${IP_ADDRESS} '
                        cd i27-helpdesk-ui && \
                        docker build -t ${IMAGE_NAME} --build-arg NEXT_PUBLIC_API_BASE_URL=http://${IP_ADDRESS}:8080 .
                    '
                """
            }
        }
        stage('image run'){
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no ${USER_NAME}@${IP_ADDRESS} '
                        docker run -d --name ${CONTAINER_NAME} -p 3000:3000 ${IMAGE_NAME}
                    '
                """
            }
        }
    }
}
