pipeline{
    agent any 
    
    tools {
        jdk 'java-17'
        maven 'maven'
    }
    environment {
        IMAGE_NAME = "afeef96867/itkannadigaru-blogpost:${GIT_COMMIT}"
        AWS_REGION = "us-west-2"
        CLUSTER_NAME = "itkannadigaru-cluster"
        NAMESPACE = "microdegree"
    }
    
    stages {
        stage ('git-checkout') {
            steps {
               git url: 'https://github.com/afeef96867-hub/ITKannadigaru-Java-based-app.git',branch:'prod1'
            }
        }
        stage ('compile') {
            steps {
                sh '''
                mvn compile
                '''
            } 
        }
        stage ('packaging') {
            steps {
                sh '''
                mvn clean package
                '''
            }
        }
        stage ('docker-build') {
            steps {
                sh '''
                printenv
                docker build -t ${IMAGE_NAME} .
                '''
            }
        }
        //stage ('docker-testing') {
           // steps {
           //     sh '''
             //   docker rm -f itkannadigaru-blogpost-test || true
            //    docker run -t -d --name itkannadigaru-blogpost-test -p 9000:8080 ${IMAGE_NAME} 
            //    '''
            //}
        //}
        stage('login to dockserhub'){
            steps{
                script{
                       withCredentials([usernamePassword(credentialsId: 'docker-hub-creds',usernameVariable:'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                       sh '''echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'''
                    }
                }
            }
        }
        stage('push to dockerhub'){
            steps{
                sh'''
                docker push ${IMAGE_NAME}
                '''
            }
        }

        stage('update the k8 cluster'){
            steps{
                script{
                   sh "aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}"     
                }
            }
        }

        stage('Deploy to EKS cluster'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: ' itkannadigaru-cluster', contextName: '', credentialsId: 'kube', namespace: 'microdegree', restrictKubeConfigAccess: false, serverUrl: 'https://698A936E405D66950C9BB36F5A1DDC21.gr7.us-west-2.eks.amazonaws.com') {
                    sh " sed -i 's|replace|${IMAGE_NAME}|g' deployment.yml "
                    sh " kubectl apply -f deployment.yml -n ${NAMESPACE}"
                }
            }
        }
        
        stage('Connect to EKS') {
            steps {
                sh """
                aws eks update-kubeconfig \
                --region ${AWS_REGION} \
                --name ${CLUSTER_NAME}
                """
            }
        }

        stage('verify'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: 'itkannadigaru-cluster', contextName: '', credentialsId: 'kube', namespace: 'microdegree', restrictKubeConfigAccess: false, serverUrl: 'https://5C8AAAF39774CCEADBBFDBA167D06AE3.sk1.us-west-2.eks.amazonaws.com'){
                    sh " kubectl get pods -n microdegree"
                    sh " kubectl get svc -n ${NAMESPACE}"
                }
            }
        }   
    }
}
