pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                // Checkout source code from your version control system (e.g., Git)
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/master']], // Change the branch as needed
                          userRemoteConfigs: [[url: 'https://github.com/loharaws/nodejs-todo-cicd.git']]])
            }
        }

        stage('Build') {
            steps {
                script {
                    sshagent(['nodejs-agent']) {
                          sh "scp -r /var/lib/jenkins/workspace/js-pipeline/* ansible@172.31.36.123:/home/ansible/nodejs/"
                          sh "ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/id_rsa ansible@172.31.36.123 'docker build -t nodejs-app /home/ansible/nodejs/.'"
                        // ... other steps
                    }
                }
            }
        }
        stage("Push to DockerHub") {
            steps {
                   echo "pushing the image to Docker hub"
                   withCredentials([usernamePassword(credentialsId: "DockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                   
                   sh "ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/id_rsa ansible@172.31.36.123 'docker tag nodejs-app ${env.dockerHubUser}/nodejs-app:latest'"
                   
                   sh "ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/id_rsa ansible@172.31.36.123 'docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}'"
                   
                   sh "ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/id_rsa ansible@172.31.36.123 docker push ${env.dockerHubUser}/nodejs-app:latest"
                   }
               }
           }
         stage('Deploy') {
             steps {
                // Deploy your application (e.g., start server)
                   sh "ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/id_rsa ansible@172.31.36.123 'cd /home/ansible/nodejs/ && docker-compose down && docker-compose up -d'"     
                
            }
        }

     }
 
}
