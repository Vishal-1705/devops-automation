pipeline {
    agent any

    // tools{
    //     maven 'maven_3_5_0'
    // }

    environment{
        DOCKERHUB_USERNAME = "vishalbasani"
        APP_NAME = "devops-integration"
        //IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
    }

    stages{
        stage('Build Maven'){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Vishal-1705/devops-automation']]])
                sh 'mvn clean install'
            }
        }
        stage('Build docker image'){
            steps{
                script{
                    sh 'sudo chmod 666 /var/run/docker.sock'
                    //sh 'docker build -t ${IMAGE_NAME} .'
                    docker_image = docker.build "${IMAGE_NAME}"
                }
            }
        }
        stage('Push image to Hub'){
            steps{
                script{
                   withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'dockerhubpwd')]) {
                   sh 'docker login -u vishalbasani -p ${dockerhubpwd}'

        }
                   //sh 'docker push ${IMAGE_NAME}'
                   docker_image.push("${env.BUILD_NUMBER}")
                   //docker_image.push("latest")
                }
            }
        }
        stage('Delete docker images'){
            steps{
                script{
                    sh "docker rmi ${IMAGE_NAME}:${env.BUILD_NUMBER}"
                    //sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
        // stage('Update kubernetes deployment file'){
        //     steps{
        //         script{
        //             sh """
        //             cat deploymentservice.yaml
        //             sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deploymentservice.yaml
        //             cat deploymentservice.yaml
        //             """
        //         }
        //     }
        // }
        // stage('Push the changed deployment file to git'){
        //     steps{
        //         script{
        //             sh """
        //             git config --global user.name "Vishal"
        //             git config --global user.email "vishalreddy304@gmail.com"
        //             git add deploymentservice.yaml
        //             git commit -m "Push the changed deployment file to git"
        //             """

        //             withCredentials([gitUsernamePassword(credentialsId: 'github-credentials', gitToolName: 'Default')]) {
        //                 sh "git pull --rebase https://github.com/Vishal-1705/devops-automation.git main"
        //                 sh "git push https://github.com/Vishal-1705/devops-automation.git main"
        //             }
        //         }
        //     }
        // }
        // stage('Deploy to k8s'){
        //     steps{
        //         script{
        //             kubernetesDeploy(configs: 'deploymentservice.yaml',kubeconfigId: 'k8sconfigpwd')
        //             //kubernetesDeploy configs: 'deploymentservice.yaml', kubeConfig: [path: ''], kubeconfigId: 'k8sconfigpwd', secretName: '', ssh: [sshCredentialsId: '*', sshServer: ''], textCredentials: [certificateAuthorityData: '', clientCertificateData: '', clientKeyData: '', serverUrl: 'https://']
        //             //sh 'kubectl apply -f deploymentservice.yaml'
        //         }
        //     }
        // }
        stage('Trigger Manifest Update'){
            steps{
                echo "Triggering manifest update file"
                build job: 'devops-automation-updatemanifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
            }
        }
    }
}
