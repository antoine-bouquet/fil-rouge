pipeline {
    environment {
        IMAGE_NAME = "antoinebouquet1010/fil-rouge-test"
        IMAGE_TAG = "latest"
        IMAGE_REPO = "antoinebouquet1010"
     }
     agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build -t $IMAGE_REPO/$IMAGE_NAME:$IMAGE_TAG .'
                }
             }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    docker run --name $IMAGE_NAME -d -p 80:80 -e PORT=80 $IMAGE_REPO/$IMAGE_NAME:$IMAGE_TAG
                    sleep 5
                 '''
               }
            }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
                    curl http://172.17.0.1 | grep -q "student list"
                '''
              }
           }
      }
      stage('Clean Container') {
          agent any
          steps {
             script {
               sh '''
                  docker rm -vf ${IMAGE_NAME}
               '''
             }
          }
     }
    stage('Push image on dockerhub') {
           agent any
           environment {
                DOCKERHUB_LOGIN = credentials('dockerhub_login_antoine')

            }

           steps {
               script {
                   sh '''
                   docker login --username ${DOCKERHUB_LOGIN_USR} --password ${DOCKERHUB_LOGIN_PSW}
                   docker push ${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                   '''
               }
           }
        }
        stage('deploy with ansible') {
            agent { docker { image 'dirane/docker-ansible:latest' } }
            steps {
                script {
                    sh '''
                        cd ansible
                        ansible-playbook -i prod.yml student.yml
                      '''
                }
            }
        }
        stage('test application') {
            agent { docker { image 'dirane/docker-ansible:latest' } }
            steps {
                script {
                    sh '''
                        cd ansible
                        ansible-playbook -i prod.yml test.yml
                      '''
                }
            }
        }
    }
}
