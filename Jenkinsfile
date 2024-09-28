pipeline{
    agent any 
    options{
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }
    environment{
        IMAGE_NAME = "${env.JOB_NAME}"
        PROJECT = 'fluted-volt-428205-p7'
        GCR_REGION = "us-west1-docker.pkg.dev"
        GCR_IMAGE_URI = "${GCR_REGION}/${PROJECT}/chereddy/${env.JOB_NAME}:${env.BUILD_NUMBER}"
        GOOGLE_APPLICATION_CREDENTIALS = credentials('mydevops')
        // GCR_PROJECT_ID = 'fluted-volt-428205-p7'
        CLUSTER_NAME = 'my-k8-cluster'
        GKE_ZONE = 'us-west1-a'
        HELM_CHART_DIR = "myservice"
        HELM_REPO_NAME = "devops"
        GCP_BUCKET = 'gs://helmrepo/myservice'
        // HELM_REPO_URL = "gs://helmrepo/myservice"
        DEPLOYMENT_NAME = "my-deployment"
        HELM_RELEASE_NAME = "Mysrevice"
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        GIT_OPS_BRANCH = 'main'
        ARGOCD_APP_NAME = 'myservice'
        argocd_token = 'https://34.105.43.92/'

    }
       
   stages{
        
        stage("Aauthenticate with GCP"){
            steps{
                script{
                    sh 'gcloud config set project ${PROJECT}'
                    sh 'gcloud auth activate-service-account --key-file=/opt/devops.json'
                    sh 'gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${GKE_ZONE} --project ${PROJECT}'
                }
            }
            
        }
        // stage("Creating Helm charts"){
        //     steps{
        //         script{
        //             sh 'helm create ${HELM_CHART_DIR}'

        //         }
        //     }
        // }
        stage('Build docker Image'){
            steps{
                script{
                    sh 'docker build -t ${GCR_IMAGE_URI} .'
            }
        }
        }
        stage("login and push the image to the docker hub"){
            steps{
                script{
                    
                    sh 'gcloud auth configure-docker ${GCR_REGION} '
                    sh 'docker push ${GCR_IMAGE_URI}'
                    }
                }
            }
        stage("pushing the changes back to github"){
            steps{
                script{
                   sh """
                         rm -rf argocd
                         git clone https://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@github.com/chereddynag/argocd.git
                         cd argocd/k8
                         sed -i 's|image: .*|image: ${GCR_IMAGE_URI}|g' deployment.yaml
                         git commit -am "Update image to  ${GCR_IMAGE_URI}"
                         git config user.name "chereddynag"
                         git config user.email "nagarjuna.chereddy@gmail.com"
                        //  git remote set-url origin https://github.com/chereddynag/argocd.git
                         git push origin main
                      """
                    
                }
            }
        }
        stage("trigger the argocd sync job"){
            steps{
                script{
                    echo "Triggering Argo CD sync for application: ${ARGOCD_APP_NAME}"
                     withCredentials([string(credentialsId: 'argocd_token', variable: 'TOKEN')]) {
                        sh """
                        curl -k -X POST ${ARGOCD_SERVER}/api/v1/applications/${ARGOCD_APP_NAME}/sync \
                          -H "Authorization: Bearer ${TOKEN}" \
                          -H "Content-Type: application/json"
                        """
                }
            }
        }
        
        
    }
}
}