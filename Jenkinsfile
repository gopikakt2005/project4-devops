pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '265183279684'
        ECR_REPOSITORY = '265183279684.dkr.ecr.us-east-1.amazonaws.com/project4-devops'

        SONAR_PROJECT_KEY = 'project4'

        GITOPS_REPO_URL = 'https://github.com/OWNER/GITOPS-REPO.git'
        GITOPS_BRANCH = 'main'
        GITOPS_MANIFEST = 'k8s/deployment.yaml'
    }

    stages {

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonarqube'

                    withSonarQubeEnv('sonarscanner') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.sources=project4
                        """
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    env.IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}"

                    sh '''
                        docker build \
                        -f project4/Dockerfile \
                        -t "$IMAGE_URI:$BUILD_NUMBER" \
                        project4
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker run --rm "$IMAGE_URI:$BUILD_NUMBER" python manage.py check
                    docker run --rm "$IMAGE_URI:$BUILD_NUMBER" python manage.py test
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region "$AWS_REGION" |
                    docker login --username AWS --password-stdin \
                    "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"

                    docker push "$IMAGE_URI:$BUILD_NUMBER"
                '''
            }
        }

        stage('Deployment - ArgoCD GitOps') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-credentials',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x

                        REPO_PATH="${GITOPS_REPO_URL#https://}"

                        git clone --branch "$GITOPS_BRANCH" \
                        "https://${GIT_USER}:${GIT_TOKEN}@${REPO_PATH}" gitops

                        cd gitops

                        sed -i "0,/image:/s|image:.*|image: $IMAGE_URI:$BUILD_NUMBER|" \
                        "$GITOPS_MANIFEST"

                        git config user.name "Jenkins"
                        git config user.email "jenkins@example.com"

                        git add "$GITOPS_MANIFEST"
                        git commit -m "Deploy Project4 build $BUILD_NUMBER"
                        git push origin "$GITOPS_BRANCH"
                    '''
                }

                echo 'GitOps repository updated. ArgoCD will sync EKS.'
            }
        }
    }

    post {
        success {
            echo 'Project4 CI/CD Pipeline SUCCESS!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
