pipeline {
    agent {label 'master'}
    environment{
        AWS_DEFAULT_REGION= "us-west-2"
        AWS_ACCOUNT_ID= "899259776368"
        ECR_REPOSITORY= "my-nodeapp"
        IMAGE_TAG= "1.0.${BUILD_NUMBER}"
        ECR_URI = "public.ecr.aws/u3u6l5m8/my-nodeapp"
        SONAR_HOME= tool "Sonar"
        }

    stages {
        stage('Clean Workspace'){
            steps{
                cleanWs()
                echo 'Clean Workspace completed..................**>>>>>>>>>>'
            }
        }
        
        stage('Code clon from Github') {
            steps {
                git url: "https://github.com/Omsonawale/node-todo-cicd.git", branch: "master" 
                echo 'Code clone completed..................!*'
            }
        }
        stage('SonarQube Quality Analysis') {
            steps {
                withSonarQubeEnv("Sonar"){
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=Todo-node-app -Dsonar.projectKey=Todo-node-app"
                }
                echo 'Sonar Scan Completed..................*'
            }
        }
        
        stage('Authonticate with AWS and ECR'){
            steps{
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials-id']
                    ]){
                    
                    sh'''
                        echo "Authenticating with AWS..."
                        export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                        export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                        export AWS_DEFAULT_REGION=${AWS_DEFAULT_REGION}
                        
                        aws sts get-caller-identity
                        
                        echo "Logging into ECR >>>>>>>>>>>"
                        aws ecr-public get-login-password --region us-east-1 |
                        docker login --username AWS --password-stdin public.ecr.aws/u3u6l5m8
                        '''
                }
            }
        }
        stage('Build TAG and Push to ECR ') {
            steps {
    
             sh'''
               
                echo 'Building image..................**'
                docker build -t my-nodeapp:latest .
                
                echo 'Tagging image with ${IMAGE_TAG} and latest...' 
                docker tag my-nodeapp:latest ${ECR_URI}:${IMAGE_TAG}
                docker tag my-nodeapp:latest ${ECR_URI}:latest  
                
                echo "Pushing images to ECR..........>>>>>>"
                docker push ${ECR_URI}:${IMAGE_TAG}
                docker push ${ECR_URI}:latest
                
                
                echo 'Build  and Tagging Completed..................**'
                '''
            }
        }
        
         stage('Scan Latest Docker Image using Trivy') {
            steps {
                sh "trivy image ${ECR_URI}:${IMAGE_TAG}"
            }
        }
        
         stage('OWASP Dependancy Check') {
            steps {
                //dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
               // dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                echo 'OWASP Dependency Check Completed.................**'
            }
        }
        stage("Cleaning Exesting Container"){
            steps{
                sh'''
                        if [ $(docker ps -aq -f name=myapp-cointainer) ]; then
                docker rm -f myapp-cointainer
            else
                echo "No existing container found"
            fi
                    '''
                    
            }
        }
         
        stage('creating cointainer') {
            steps {
                sh 'docker run -d -p 8000:8000 --name myapp-cointainer  my-nodeapp:latest '
                echo 'Cointainer created.....................!**'
            }
        }
    }
}

