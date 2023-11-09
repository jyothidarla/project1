# Build CI / CD Pipeline using Jenkins and deploy the real world Web Application in AWS Cloud

Technologies Used:-
-------------------
1. Jenkins
2. Groovy
3. AWS Cloud
4. Git
5. Docker  

Pre-Requiset:
--------------
* Setup Jenkins (ubuntu with t2.large)
* setup webhook integration for automatic Build Triggers 
* Create A Role (name: ecr_role1 ) with AdministrationAccess  Attach to Jenkins Machine  
* Create ECR Repo in the same region where jenkins machine available For to upoad image into that repo 

jenkinsFile Code steps
-------------------------------
>step1: To build pipeline code if we have any agent machine we need to include here To build job From agent mahine 
   
    pipeline{
        agent any
    }

>step2: To Pass environments in jenkins

    environment {
        APP_NAME = "mypp"
        RELEASE = "1.0.0"
        ECR_REGISTRY = "721346825573.dkr.ecr.us-east-1.amazonaws.com"
        ECR_REPOSITORY = "thej"
        IMAGE_NAME = "${ECR_REGISTRY}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }

>step3: clean workspace 

        stage("Clear Workspace"){
            steps{
                cleanWs()
            }
        }

>step4: Download Code From Repo

        stage("Download Code"){
            steps{
                git 'https://github.com/thej950/project-1.git'
            }
        }

>step5: Build and test application code using maven 

        stage("Test Application"){
            steps{
                    sh 'mvn test'
            }
        }
        stage("Build Application Code"){
            steps{
                sh 'mvn package'
            }
        }

>step6: Build the Docker image 
    
* First Download Docker into Jenkins Machine for Building image from Dockerfile
       
        # sudo curl -fsSL https://get.docker/com -o docker.sh
        # sh docker.sh

* Add jenkins user into dokcer group to build docker images Directly 
        # sudo usermod -a -G docker jenkins
* Add jenkins user into sudo group for to run root commands also enter details of jenkins user in sudoers file To avoid to no to ask password [ jenkins ALL (ALL:ALL) NOPASSWD:ALL ]
       
        # sudo usermod -a -G sudo jenkins
        # vim /etc/sudoers
        
* To Build Docker image    
        
        stage("Build Docker image"){
            steps{
                sh  'sudo docker build -t $APP_NAME .'
            } 
        }

>step7: To push Docker image into ECR repository 

* there are combinations of steps included here To push already existing dokcer image into ecr repository
* check ecr repo is avialabile if its not create first in AWS ECR service 
* take commands after creating ecr repo include into code to upload image into ecr repo 
* Below are jenkins code To upload image into ecr repo 

        stage("Push Docker image To ECR"){
            steps{
                script{
                    sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"
                    sh "docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"

                }
            }
        }


