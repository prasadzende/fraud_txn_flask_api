pipeline {
    agent any

    stages {
        stage('Verify Branch') {
            steps {
                echo "$GIT_BRANCH"
            }
        }
        stage('Start build & test app') {
            steps {
                sh(script:"""
                    docker-compose up -d
                    ./response-check.sh
                """)
            }
            post {
                success {
                    echo "App Started successfully :)"
                }
                failure {
                    echo "App failed to start :("
                }
            }
            
        }
        stage('Stop test app') {
            steps {
               
                sh(script:"""
                    docker-compose down
                """)
            }
            
        }
        stage('Push container') {
            steps {
               echo "Workspace is $WORKSPACE"
               dir("$WORKSPACE"){
                   script{
                       docker.withRegistry("https://index.docker.io/v1/","Dockerhub_Creds"){
                           def image = docker.build('prasadzende/sample_repo:credit_cls_demo')
                           image.push()
                       }
                   }
               }
            }
        }
        stage('Deploy to Test') {
            environment {
               ENVIRONMENT = 'test'
            }
            steps {
                echo "Deploying to ${ENVIRONMENT}"
                script {
                    kubernetesDeploy(
                        configs: "k8s-deployment.yml",
                        kubeconfigId: "K8S_Config",
                        enableConfigSubstitution: true
                    )
                }
            }            
        }
    }
}
