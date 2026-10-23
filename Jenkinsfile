pipeline {
    agent any

    environment {
        SERVER = '172.31.43.249'
        DEPLOY_DIR = '/home/ubuntu/java-deploy'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Deploy Java Application') {
            steps {
                sshagent(credentials: ['java-deploy-ssh1']) {
                    sh '''
                        set -e

                        tar --exclude=.git -czf /tmp/java-app.tar.gz .

                        scp -o StrictHostKeyChecking=yes \
                          /tmp/java-app.tar.gz \
                          ubuntu@$SERVER:/tmp/java-app.tar.gz

                        ssh -o StrictHostKeyChecking=yes ubuntu@$SERVER \
                          "mkdir -p $DEPLOY_DIR"

                        ssh -o StrictHostKeyChecking=yes ubuntu@$SERVER \
                          "tar -xzf /tmp/java-app.tar.gz -C $DEPLOY_DIR"

                        ssh -o StrictHostKeyChecking=yes ubuntu@$SERVER \
                          "cd $DEPLOY_DIR && docker build -t java-app:latest ."

                        ssh -o StrictHostKeyChecking=yes ubuntu@$SERVER \
                          "docker rm -f java-app || true"

                        ssh -o StrictHostKeyChecking=yes ubuntu@$SERVER \
                          "docker run -d \
                            --name java-app \
                            --restart unless-stopped \
                            -p 127.0.0.1:5000:5000 \
                            java-app:latest"

                        ssh -o StrictHostKeyChecking=yes ubuntu@$SERVER \
                          "curl --fail --retry 10 \
                            --retry-delay 3 \
                            --retry-connrefused \
                            http://localhost:5000/"
                    '''
                }
            }
        }
    }
}
