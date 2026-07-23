pipeline {
    agent any

    environment {
        AWS_REGION = 'eu-north-1'
        ACCOUNT_ID = '450444046629'
        ECR_REPO = 'user29-petclinic'
        IMAGE_TAG = "${BUILD_NUMBER}"
        AWS_SHARED_CREDENTIALS_FILE = '/var/jenkins_home/.aws/credentials'
        AWS_PAGER = ''
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                trivy fs .
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t $ECR_REPO:$IMAGE_TAG .
                docker tag $ECR_REPO:$IMAGE_TAG $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                docker push $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }

	stage('Update Manifest') {
	    steps {
        	withCredentials([usernamePassword(
            		credentialsId: 'github-token',
            		usernameVariable: 'GIT_USER',
            		passwordVariable: 'GIT_TOKEN'
        	)]) {

            	sh '''
            	sed -i "s|image: .*|image: $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG|" k8s/02_deployment.yaml

            	git config user.email "jenkins@example.com"
            	git config user.name "jenkins"

            	git add k8s/02_deployment.yaml

            	git commit -m "Update image to build $IMAGE_TAG" || true

           	git push https://$GIT_USER:$GIT_TOKEN@github.com/simmmba/petclinic-cicd.git HEAD:main
            	'''
        	}
    		}
	}
    }
}
