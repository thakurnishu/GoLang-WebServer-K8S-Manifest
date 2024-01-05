@Library('Jenkins-Shared-Lib') _

pipeline{
    agent any

    parameters{
        string(name: 'ImageTag')
    }

    environment{
        AZURE_SUBSCRIPTION_ID = credentials('subscription_id')
        AZURE_TENANT_ID = credentials('tenant_id')
        SERVICE_PRINCIPAL_ID = credentials('principal_id')
        SERVICE_PRINCIPAL_PASSWORD = credentials('principal_password')
        STORAGE_KEY = credentials('azure_storage_key')
        GITHUB_EMAIL= credentials('github_email')
        GITHUB_CRED_ID= 'github'
        IMAGENAME = 'gowebserver'
        GITHUB_USER= 'thakurnishu'
    }

    stages{
        stage('Clean WorkSpace'){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        stage('Git Checkout'){
            steps{
                script{
                    gitCheckout(
                        branch: "main",
                        url: "https://github.com/thakurnishu/GoLang-WebServer-K8S-Manifest.git"
                    )
                }
            }
        }
        stage('Terraform Initialization'){
            steps{
                script{
                    terraInit(STORAGE_KEY)
                }
            }
        }
        stage('Terraform Plan'){
            steps{
                script{
                    terraPlan(
                        SUBSCRIPTION_ID: AZURE_SUBSCRIPTION_ID,
                        TENANT_ID: AZURE_TENANT_ID,
                        PRINCIPAL_ID: SERVICE_PRINCIPAL_ID,
                        PRINCIPAL_PASSWORD: SERVICE_PRINCIPAL_PASSWORD
                    )
                }
            }
        }
        stage('Terraform Apply'){
            steps{
                script{
                    terraApply()
                }
            }
        }
        stage('Connecting to Kubernetes Cluster'){
            steps{
                script{
                    connectToK8S()
                }
            }
        }
        stage('Installing ArgoCD'){
            steps{
                script{
                    sh """
                        chmod +x InstallArgoCD.sh
                        sh InstallArgoCD.sh
                    """
                }
            }
        }
        stage('Update Manifest Files: K8S'){
            steps{
                script{
                    def apply = false
                    try{
                        input message: 'please confirm to update manifest', ok: 'Ready to update the manifest ?'
                        apply = true
                    }catch(err){
                        apply= false
                        currentBuild.result  = 'UNSTABLE'
                    }
                    if (apply){
                        updateManifestK8S(
                            imageName: IMAGENAME,
                            imageTag: "${params.ImageTag}" 
                        ) 
                    }
                }
            }
        }
        stage('Push Manifest Files: Github'){
            steps{
                script{
                    def gitHubURL = "https://github.com/thakurnishu/GoLang-WebServer-K8S-Manifest.git"
                    def requiredURL = gitHubURL.replace("https://", "").replace(".git", "")
                    pushManifestGithub(
                        githubUserName: GITHUB_USER,
                        githubEmail: GITHUB_EMAIL,
                        imageTag: "${params.ImageTag}",
                        githubCredID: GITHUB_CRED_ID,
                        githubURL: requiredURL
                    )
                }
            }
        }
    }
}