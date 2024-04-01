
pipeline {
    agent any
    environment {
        SSH_KEY = credentials('ssh-credentials-id') // Thay 'your-ssh-key-id' bằng ID của SSH key trong Jenkins Credentials
        HOST = 'ec2-18-143-167-76.ap-southeast-1.compute.amazonaws.com' // Thay 'your-ec2-hostname-or-ip' bằng địa chỉ hostname hoặc IP của EC2 instance
        USERNAME = 'root' // Thay 'your-ssh-username' bằng tên người dùng SSH trên EC2 instance
        IMAGE_NAME = 'danhdz1310/my-repo:latest' // Thay 'username/image_name:tag' bằng thông tin của image Docker Hub bạn muốn triển khai
        CONTAINER_PORT = '1310' // Cổng container mà ứng dụng của bạn đang lắng nghe
        HOST_PORT = '1310' // Cổng trên EC2 instance mà bạn muốn ánh xạ với cổng của container
    }
    

    tools {
        dockerTool 'docker'
    }

    stages {
        stage("pull repo") {
            steps {
                git branch: 'main', credentialsId: 'ssh-credentials-id', url: 'https://github.com/trivien0/devops-todo-app.git'
                sh 'ls -la'
            }
        }
        stage("build and test") {
            steps {
               sh 'echo "FROM node:20-alpine" > Dockerfile'
               sh 'echo "RUN mkdir -p /home/node/app/node_modules && chown -R node:node /home/node/app" >> Dockerfile'
               sh 'echo "WORKDIR /home/node/app" >> Dockerfile'
               sh 'echo "COPY package*.json ./" >> Dockerfile'
               sh 'echo "RUN npm install" >> Dockerfile'
               sh 'echo "COPY  . ." >> Dockerfile'
               sh 'echo "EXPOSE 8000" >> Dockerfile'
               sh 'echo \'CMD [ "node", "app.js" ]\' >> Dockerfile'
               sh 'docker build -t danhdz1310/my-repo .'
               
            }
        }
        stage("Docker login and push docker image") {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-password', variable: 'dockerhubpwd')]) {
                    sh 'docker login -u danhdz1310 -p ${dockerhubpwd}'
                }
                sh 'docker push danhdz1310/my-repo'
            }
        }
        stage('Deploy to EC2') {
            steps {
                script {
                    // Đảm bảo SSH key có quyền truy cập
                    sshagent(['ssh-credentials-id']) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no -i $SSH_KEY $USERNAME@$HOST << EOF
                            sudo docker pull $IMAGE_NAME
                            sudo docker run -d -p $HOST_PORT:$CONTAINER_PORT --name my_container $IMAGE_NAME
                            docker ps
                            EOF
                        '''
                    }
                }
            }
        }
     
    }
}

