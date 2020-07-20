pipeline {
     agent any
     stages {
         stage('Build') {
             steps {
                 sh 'echo "Hello World"'
                 sh '''
                     echo "Multiline shell steps works too"
                     ls -lah
                 '''
             }
         }
         stage('Lint HTML') {
              steps {
                  sh 'tidy -q -e *.html'
              }
         }
         stage('Build Docker Image') {
              steps {
                  sh "docker build -t capstone-cloud-devops ."
              }
         }
         stage('Push Docker Image') {
              steps {
                  withDockerRegistry([url:"", credentialsId: "dockerhub"]) {
                      sh "docker tag capstone-cloud-devops rawanalkhalawi/capstone-cloud-devops"
                      sh "docker push rawanalkhalawi/capstone-cloud-devops"
                  }
              }
         }
         stage('Security Scan') {
              steps { 
                 aquaMicroscanner imageName: 'alpine:latest', notCompliesCmd: 'exit 1', onDisallowed: 'fail', outputFormat: 'html'
               }
         }         
         stage('Upload to AWS') {
              steps {
                  withAWS(region:'us-east-2',credentials:'aws-static') {
                  sh 'echo "Uploading content with AWS creds"'
                    //   s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file:'index.html', bucket:'static-jenkins-pipline')
                    sh "aws eks --region us-east-2 update-kubeconfig --name cluster"
                    sh "kubectl config use-context arn:aws:ecs:us-east-2:773751258356:cluster/cluster"
                    sh "kubectl set image deployments/capstone-cloud-devops capstone-cloud-devops=rawanalkhalawi/capstone-cloud-devops:latest"
                    sh "kubectl apply -f rollingDeployment.yml"
                    sh "kubectl get nodes"
                    sh "kubectl get deployment"
                    sh "kubectl get pod -o wide"
                    sh "kubectl get service/capstone-cloud-devops"
                  }
              }
         }
     }
}