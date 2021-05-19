pipeline {
    environment {
        IMAGE_NAME = "fil-rouge"
        IMAGE_TAG = "latest"
        IMAGE_REPO = "registry.gitlab.com/fil-rouge2/fil-rouge"
        LOCAL_REPO = "192.168.31.135:5001"
     }
     agent none
     stages {
         stage('Build image') {
             agent { docker { image 'docker' } }
             steps {
                script {
                  sh 'docker build -t ${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG} simple_api/'
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
        stage('Clean API Container') {
          agent { docker { image 'docker' } }
          steps {
             script {
               sh '''
                  docker rm -vf ${IMAGE_NAME}
               '''
             }
          }
      }

        stage('Run containers of the  application') {
            agent { docker { image 'tmaier/docker-compose' } }
            steps {
               script {
                 sh '''
                    docker-compose up -d
                    sleep 5
                 '''
                }
            }
       }

       stage('Test the application') {
            agent { docker { image 'curlimages/curl' } }
            steps {
                script {
                        sh '''
                        curl http://192.168.31.135:50 
                        '''
              }
           }
        }
      stage('Clean Container') {
          agent { docker { image 'docker' } }
          steps {
             script {
               sh '''
                  docker rm -vf api website
               '''
             }
          }
     }
    stage('Push image on gitlab') {
           agent { docker { image 'docker' } }
           environment {
                GITLAB_LOGIN = credentials('gitlab_login_antoine')
            }

           steps {
               script {
                   sh '''
                   docker login registry.gitlab.com --username ${GITLAB_LOGIN_USR} --password ${GITLAB_LOGIN_PSW}
	           docker push ${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                   '''
               }
           }
        }
        stage('deploy with ansible') {
            agent { docker { image 'dirane/docker-ansible:latest' } }
            environment {
                GITLAB_LOGIN = credentials('gitlab_login_antoine')
                API_LOGIN = credentials('student_api_login')
            }

            steps {
                script {
                    sh '''
                        cd ansible
                        ansible-playbook  -i prod.yml -e "username=${GITLAB_LOGIN_USR} password=${GITLAB_LOGIN_PSW} apiuser=${API_LOGIN_USR} apipwd=${API_LOGIN_PSW}"  student.yml
                      '''
                }
            }
        }
       stage('test the deployment of the front end ') {
            agent { docker { image 'dirane/docker-ansible:latest' } }
            steps {
                script {

                    sh '''
                        cd ansible
                        ansible-playbook -i prod.yml test_front.yml
                        '''
                }
            }
        }
       stage('test the deployment of the API ') {
            agent { docker { image 'dirane/docker-ansible:latest' } }
            steps {
                script {

                    sh '''
                        cd ansible
                        ansible-playbook -i prod.yml test_api.yml
                        '''
                }
            }
        }
    }
}
