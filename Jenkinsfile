pipeline {
    environment {
        IMAGE_NAME = "fil-rouge-test"
        IMAGE_TAG = "latest"
        IMAGE_REPO = "antoinebouquet1010"
     }
     agent none
     stages {
         stage('Build image') {
             agent { docker { image 'docker' } }
             steps {
                script {
                  sh 'docker build -t $IMAGE_REPO/$IMAGE_NAME:$IMAGE_TAG simple_api/'
                }
             }
        }
        stage('Run container based on builded image') {
            agent { docker { image 'docker' } }
            steps {
               script {
                 sh '''
                    docker run --name $IMAGE_NAME -d -p 5000:5000 $IMAGE_REPO/$IMAGE_NAME:$IMAGE_TAG
                    sleep 5
                 '''
               }
            }
       }
       stage('Test image api') {
            agent { docker { image 'curlimages/curl' } }
            steps {
                script {
                        sh '''
                        curl -u toto:python -X GET http://192.168.31.135:5000/pozos/api/v1.0/get_student_ages 
                        '''
              }
           }
        }
      stage('Clean Container') {
          agent { docker { image 'docker' } }
          steps {
             script {
               sh '''
                  docker rm -vf ${IMAGE_NAME}
               '''
             }
          }
     }
    stage('Push image on dockerhub') {
           agent { docker { image 'docker' } }
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
                        echo "$PWD"
                        ls
                        cd ansible
                        cd ../
                        echo "$PWD"
                        cd ansible
                        ansible-playbook  -i prod.yml student.yml
                      '''
                }
            }
        }
    }
}
