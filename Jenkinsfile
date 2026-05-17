pipeline {
    agent {
        label 'slave2'
    }

    environment {
        USER_NAME  = "phani"
        IP_ADDRESS = "34.142.188.216"
    }

    stages {

        stage('Connecting to VM') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no ${USER_NAME}@${IP_ADDRESS} \
                    'echo Connected to test-vm'
                """
            }
        }

        stage('UI Build & Deploy') {

            environment {
                IMAGE_NAME     = "i27-helpdesk-ui:dev"
                CONTAINER_NAME = "i27-ui"
            }

            steps {

                sh """
                    ssh -o StrictHostKeyChecking=no ${USER_NAME}@${IP_ADDRESS} '

                        rm -rf i27-helpdesk-ui && \
                        git clone https://github.com/i27academy/i27-helpdesk-ui.git && \
                        cd i27-helpdesk-ui && \

                        docker rm -f ${CONTAINER_NAME} || true && \
                        docker rmi ${IMAGE_NAME} || true && \

                        docker build -t ${IMAGE_NAME} --build-arg NEXT_PUBLIC_API_BASE_URL=http://${IP_ADDRESS}:8080 . && \

                        docker run -d --name ${CONTAINER_NAME} -p 3000:3000 ${IMAGE_NAME}
                    '
                """
            }
        }
        stage('Gateway'){

            environment {
                IMAGE_NAME     = "i27-helpdesk-gateway:dev"
                CONTAINER_NAME = "i27-gateway"
            }
            steps {

                sh """
                    ssh -o StrictHostKeyChecking=no ${USER_NAME}@${IP_ADDRESS} '

                        rm -rf i27-helpdesk-gateway && \
                        git clone https://github.com/i27academy/i27-helpdesk-gateway.git && \

                        cd i27-helpdesk-gateway && \
                                        cat > .env.dev <<EOF
SERVER_PORT=8080
UI_ORIGIN=http://${IP_ADDRESS}:3000
AUTH_SERVICE_URL=http://${IP_ADDRESS}:8081
TICKET_SERVICE_URL=http://${IP_ADDRESS}:8082
COMMENT_SERVICE_URL=http://${IP_ADDRESS}:8083
ATTACHMENT_SERVICE_URL=http://${IP_ADDRESS}:8084

JWT_SECRET=i27academy-secret-key-which-is-32chars
EOF


                        docker rm -f ${CONTAINER_NAME} || true && \
                        docker rmi ${IMAGE_NAME} || true && \

                        docker build -t ${IMAGE_NAME} . && \

                        docker run -d --name ${CONTAINER_NAME} --env-file .env.dev -p 8080:8080 ${IMAGE_NAME}
                    '
                """
            }

        }
        stage('Auth Build & Deploy'){
            environment {
                IMAGE_NAME     = "i27-helpdesk-auth:dev"
                CONTAINER_NAME = "i27-auth"
                
            }
            steps {

                sh """
                    ssh -o StrictHostKeyChecking=no ${USER_NAME}@${IP_ADDRESS} '

                        rm -rf i27-helpdesk-auth-service && \
                        git clone https://github.com/i27academy/i27-helpdesk-auth-service.git && \
                        cd i27-helpdesk-auth-service && \
                        cat > .env.dev <<EOF
SERVER_PORT=8081
DB_JDBC_URL=jdbc:mysql://34.172.178.28:3306/helpdesk_dev
DB_USERNAME=helpdesk_user
DB_PASSWORD=Helpdesk@123

JWT_SECRET=i27academy-secret-key-which-is-32chars
JWT_EXPIRY_MILLIS=3600000

EOF

                         docker rm -f ${CONTAINER_NAME} || true && \
                        docker rmi ${IMAGE_NAME} || true && \
                        
                        docker build -t ${IMAGE_NAME} . && \

                        docker run -d --name ${CONTAINER_NAME} --env-file .env.dev -p 8081:8081 ${IMAGE_NAME}
                    '
                """
            }

        }
        stage('Ticket Build & Deploy'){
            environment {
                IMAGE_NAME     = "i27-helpdesk-ticket:dev"
                CONTAINER_NAME = "i27-ticket"
               
            }
            steps {

                sh """
                    ssh -o StrictHostKeyChecking=no ${USER_NAME}@${IP_ADDRESS} '

                        git clone https://github.com/i27academy/i27-helpdesk-ticket-service.git && \
                        cd i27-helpdesk-ticket-service && \

                        cat > .env.dev <<EOF
SERVER_PORT=8082
DB_JDBC_URL=jdbc:mysql://34.172.178.28:3306/helpdesk_dev
DB_USERNAME=helpdesk_user
DB_PASSWORD=Helpdesk@123

NOTIFICATION_SERVICE_URL=http://34.172.178.28:3306:8086
EOF

                         docker rm -f ${CONTAINER_NAME} || true && \
                        docker rmi ${IMAGE_NAME} || true && \
                        
                        docker build -t ${IMAGE_NAME} . && \

                        docker run -d --name ${CONTAINER_NAME} --env-file .env.dev -p 8082:8082 ${IMAGE_NAME}
                    '
                """
            }

        }
        stage('Comment Build & Deploy'){
            environment {
                IMAGE_NAME     = "i27-helpdesk-comment:dev"
                CONTAINER_NAME = "i27-comment"
                
            }
            steps {

                sh """
                    ssh -o StrictHostKeyChecking=no ${USER_NAME}@${IP_ADDRESS} '

                        git clone https://github.com/i27academy/i27-helpdesk-comment-service.git && \
                        cd i27-helpdesk-comment-service && \

                        cat > .env.dev <<EOF
DB_USER=helpdesk_user
DB_PASSWORD=Helpdesk@123
DB_HOST=34.172.178.28
DB_PORT=3306
DB_NAME=helpdesk_dev

NOTIFICATION_URL=http://${IP_ADDRESS}:8084/notifications/event
TICKET_SERVICE_URL=http://${IP_ADDRESS}:8082/tickets
EOF

                         docker rm -f ${CONTAINER_NAME} || true && \
                        docker rmi ${IMAGE_NAME} || true && \
                        
                        docker build -t ${IMAGE_NAME} . && \

                        docker run -d --name ${CONTAINER_NAME} --env-file .env.dev -p 8083:8083 ${IMAGE_NAME}
                    '
                """
            }

        }

    }
}
