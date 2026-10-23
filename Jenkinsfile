
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
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: ['java-deploy-ssh1']) {
                    sh '''
                        set -e

                        ssh -o StrictHostKeyChecking=yes ubuntu@$SERVER '
                            echo "Checking Java container..."
                            docker ps -a --filter name=java-app

                            echo "Waiting for Java application..."

                            for i in $(seq 1 30); do
                                if curl --fail --silent --show-error \
                                    --max-time 5 \
                                    http://127.0.0.1:5000/; then
                                    echo ""
                                    echo "Java application is running successfully!"
                                    exit 0
                                fi

                                echo "Attempt $i failed. Retrying in 5 seconds..."
                                sleep 5
                            done

                            echo "Java application health check failed."
                            echo "Container logs:"
                            docker logs --tail 100 java-app
                            exit 1
                        '
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Java Application Deployment SUCCESS!'
        }

        failure {
            echo 'Java Application Deployment FAILED. Check console output.'
        }
    }
}
