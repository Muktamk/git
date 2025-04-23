pipeline {
    agent any
        parameters {
        string(name: 'BRANCH_NAME', defaultValue: '', description: 'Enter the branch name to deploy')
    }
    environment {
        EKS_CLUSTER_NAME = "dev-2024-IDI-AP-EKSCluster"
        ECR_REPO_FULL_NAME = "637423197082.dkr.ecr.us-east-2.amazonaws.com/idi-ap-backend-new"
        // AWS credentials for Jenkins
        AWS_ACCESS_KEY_ID = credentials('new-aws-credential')
        AWS_SECRET_ACCESS_KEY = credentials('new-aws-credential')
        AWS_REGION = "us-east-2"
        POD_REPLICA_COUNT = "2"
        IDI_API_URL = "dev-perpetual-api.idiinventory.com"
        INGRESS_LB_NAME_TAG = "dev-idi-ingress-LB"
        INGRESS_GROUP = "idi-dev-ingress-group"
        INGRESS_ACM_CERTIFICATE_ARN = "arn:aws:acm:us-east-2:637423197082:certificate/9f47f0fd-1f9f-40ed-a8bb-0f53a5cb1587"
        INGRESS_HOST = "dev-perpetual-api.idiinventory.com"
        DEV_SECRET_KEY = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.DEV_SECRET_KEY\'', returnStdout: true).trim()
        DEV_DB_NAME = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.DEV_DB_NAME\'', returnStdout: true).trim()
        DEV_DB_USER = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.DEV_DB_USER\'', returnStdout: true).trim()
        DEV_DB_PASSWORD = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.DEV_DB_PASSWORD\'', returnStdout: true).trim()
        DEV_DB_HOST = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.DEV_DB_HOST\'', returnStdout: true).trim()
        // Aurora Read Replica DB details
        REPLICA_DEV_DB_NAME = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.REPLICA_DEV_DB_NAME\'', returnStdout: true).trim()
        REPLICA_DEV_DB_USER = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.REPLICA_DEV_DB_USER\'', returnStdout: true).trim()
        REPLICA_DEV_DB_PASSWORD = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.REPLICA_DEV_DB_PASSWORD\'', returnStdout: true).trim()
        REPLICA_DEV_DB_HOST = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.REPLICA_DEV_DB_HOST\'', returnStdout: true).trim()
        
        K8S_NAMESPACE = "dev"
        HELM_VALUES_FILE_PATH = "infra/helm-chart/idi-ap-backend/values-generic.yaml"
        HELM_ADDITIONAL_VALUES_FILE_PATH = "infra/helm-chart/idi-ap-backend/configmap-values-dev.yaml"
        HELM_CHART_INFO_FILE_PATH = "infra/helm-chart/idi-ap-backend/Chart.yaml"
        HELM_CHART_PATH = "infra/helm-chart/idi-ap-backend"
        HELM_RELEASE_NAME = "idi-ap-backend"
        STRIPE_PUBLIC_KEY = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.STRIPE_PUBLIC_KEY\'', returnStdout: true).trim()
        STRIPE_SECRET_KEY = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.STRIPE_SECRET_KEY\'', returnStdout: true).trim()
        SECRET_KEY = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.SECRET_KEY\'', returnStdout: true).trim()
        SECRET_JWT_KEY = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.SECRET_JWT_KEY\'', returnStdout: true).trim()
        // application, aws credentials to send the email
        AWS_ACCESS_KEY = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.AWS_ACCESS_KEY\'', returnStdout: true).trim()
        AWS_SECRET_KEY = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.AWS_SECRET_KEY\'', returnStdout: true).trim()

        STRIPE_INVOICE_WEBHOOK_SECRET = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.STRIPE_INVOICE_WEBHOOK_SECRET\'', returnStdout: true).trim()
        STRIPE_INTENT_WEBHOOK_SECRET = sh(script: 'aws secretsmanager get-secret-value --secret-id idi-ap-dev --region us-east-2 --output json | jq --raw-output \'.SecretString | fromjson.STRIPE_INTENT_WEBHOOK_SECRET\'', returnStdout: true).trim()
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('checkout') {
            steps {
                script {
                    checkout scmGit(branches: [[name: "*/${params.BRANCH_NAME}"]], extensions: [], userRemoteConfigs: [[credentialsId: 'jenkins-github-connection', url: 'git@github.com:perpetualmotion/idi-ap-be.git']])
                }
            }
        }

        stage('docker-image-tag') {
            steps {
                script {
                    DOCKER_IMAGE_TAG = "V1.0.${BUILD_ID}"
                }
            }
        }

        // stage('SonarQube Analysis') {
        //     steps {
        //         script {
        //             def scannerHome = tool 'Sonarqube';
        //             withSonarQubeEnv() {
        //                 sh "${scannerHome}/bin/sonar-scanner"
        //             }
        //         }
        //     }
        // }

        stage('aws-configure') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'new-aws-credential', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    script {
                        sh "aws configure set region ${env.AWS_REGION}"
                    }
                }
            }
        }
        
        stage('Testing Access for kubectl') {
            steps {
                script {
                   sh 'aws eks update-kubeconfig --name $EKS_CLUSTER_NAME'
                }
            }
        }

        stage('Setting Env Var for commit ID to use as Image Tag') {
            steps {
                script  {
                    echo "The Docker Image tag is: ${DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin ${env.ECR_REPO_FULL_NAME}"
                }
            }  
        }

        stage('replacing the variables') {
            steps {
                script {
                    // Replacing the variables
                    sh """
                    sed -i -e 's@HELM_REPLACE_ECR_REPO_NAME@${env.ECR_REPO_FULL_NAME}@g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/HELM_REPLACE_IMAGE_TAG/${DOCKER_IMAGE_TAG}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/HELM_REPLACE_POD_REPLICA_COUNT/${env.POD_REPLICA_COUNT}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/HELM_REPLACE_IMAGE_TAG/${DOCKER_IMAGE_TAG}/g' $HELM_CHART_INFO_FILE_PATH
                    sed -i -e 's#HELM_REPLACE_BACKEND_API_URL#${env.IDI_API_URL}#g' $HELM_VALUES_FILE_PATH

                    sed -i -e 's/HELM_REPLACE_LB_NAME_TAG/${env.INGRESS_LB_NAME_TAG}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/HELM_REPLACE_INGRESS_GROUP/${env.INGRESS_GROUP}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's@HELM_REPLACE_ACM_CERTIFICATE_ARN@${env.INGRESS_ACM_CERTIFICATE_ARN}@g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's@HELM_REPLACE_INGRESS_HOST@${env.INGRESS_HOST}@g' $HELM_VALUES_FILE_PATH

                    sed -i -e 's/SECRET_HELM_REPLACE_SECRET_KEY/${env.DEV_SECRET_KEY}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/SECRET_HELM_REPLACE_DB_NAME/${env.DEV_DB_NAME}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/SECRET_HELM_REPLACE_DB_USER/${env.DEV_DB_USER}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/SECRET_HELM_REPLACE_DB_PASSWORD/${env.DEV_DB_PASSWORD}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/SECRET_HELM_REPLACE_DB_HOST/${env.DEV_DB_HOST}/g' $HELM_VALUES_FILE_PATH

                    sed -i -e 's/SECRET_HELM_REPLACE_replica_db_name/${env.REPLICA_DEV_DB_NAME}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/SECRET_HELM_REPLACE_replica_db_user/${env.REPLICA_DEV_DB_USER}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/SECRET_HELM_REPLACE_replica_db_password/${env.REPLICA_DEV_DB_PASSWORD}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/SECRET_HELM_REPLACE_replica_db_host/${env.REPLICA_DEV_DB_HOST}/g' $HELM_VALUES_FILE_PATH

                    sed -i -e 's|SECRET_HELM_REPLACE_AWS_ACCESS_KEY_ID|${AWS_ACCESS_KEY_ID.replace("'", "\\'")}|g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's|SECRET_HELM_REPLACE_AWS_SECRET_ACCESS_KEY|${AWS_SECRET_ACCESS_KEY.replace("'", "\\'")}|g' $HELM_VALUES_FILE_PATH

                    sed -i -e 's/SECRET_HELM_REPLACE_STRIPE_PUBLIC_KEY/${env.STRIPE_PUBLIC_KEY}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/SECRET_HELM_REPLACE_STRIPE_SECRET_KEY/${env.STRIPE_SECRET_KEY}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/SECRET_HELM_REPLACE_SECRET_KEY/${env.SECRET_KEY}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/SECRET_HELM_REPLACE_SECRET_JWT_KEY/${env.SECRET_JWT_KEY}/g' $HELM_VALUES_FILE_PATH

                    sed -i -e 's|SECRET_HELM_REPLACE_AWS_ACCESS_KEY|${AWS_ACCESS_KEY.replace("'", "\\'")}|g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's|SECRET_HELM_REPLACE_AWS_SECRET_KEY|${AWS_SECRET_KEY.replace("'", "\\'")}|g' $HELM_VALUES_FILE_PATH

                    sed -i -e 's/SECRET_HELM_REPLACE_STRIPE_INVOICE_WEBHOOK_SECRET/${env.STRIPE_INVOICE_WEBHOOK_SECRET}/g' $HELM_VALUES_FILE_PATH
                    sed -i -e 's/SECRET_HELM_REPLACE_STRIPE_INTENT_WEBHOOK_SECRET/${env.STRIPE_INTENT_WEBHOOK_SECRET}/g' $HELM_VALUES_FILE_PATH

                    """
                }
            }
        }

        stage('printing the env related to the stripe') {
            steps {
                script {
                    echo "Printing the values"
                    echo "The value of the STRIPE_INVOICE_WEBHOOK_SECRET is: ${env.STRIPE_INVOICE_WEBHOOK_SECRET}"
                    echo "The value of the STRIPE_INTENT_WEBHOOK_SECRET is: ${env.STRIPE_INTENT_WEBHOOK_SECRET}"
                }
            }
        }

        stage('building Docker Image') {
            steps{
                script {
                    echo "Building Docker image with tag: ${DOCKER_IMAGE_TAG}"
                    sh "docker build -t idi-ap-backend:${DOCKER_IMAGE_TAG} -f infra/Dockerfile ." 
                    sh "docker tag idi-ap-backend:${DOCKER_IMAGE_TAG} ${env.ECR_REPO_FULL_NAME}:${DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('pushing the Docker Image') {
            steps {
                script {
                    sh "docker push ${env.ECR_REPO_FULL_NAME}:${DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('deleting the docker images from the Jenkins server') {
            steps {
                script {
                    sh "docker rmi ${env.ECR_REPO_FULL_NAME}:${DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('Deploying In K8S') {
            steps {
                script {
                    sh "helm --debug upgrade --install ${env.HELM_RELEASE_NAME} ${env.HELM_CHART_PATH} -f ${env.HELM_VALUES_FILE_PATH} -f ${env.HELM_ADDITIONAL_VALUES_FILE_PATH} --namespace ${env.K8S_NAMESPACE} --create-namespace"                   
                }
            }
        }
    }
}