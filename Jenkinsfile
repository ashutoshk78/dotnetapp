pipeline {
    agent {
        node {
            label 'docker-io-ui'
        }
    }
      environment {
        jenkins_secret_username="ae029532-9015-4bd0-b7f0-d9ee65598885"
        jenkins_secret_password="5db8122c-c284-4933-b879-3391bef71e0b"
      }

    options { skipDefaultCheckout() }

    stages {
        stage ('Initialize Application Migration') {
            agent none
            steps {
                script {
                        withCredentials(
                                        [string(credentialsId: jenkins_secret_username, variable: 'SECRET_ID'),
                                        string(credentialsId: jenkins_secret_password, variable: 'SECRET_PASS')])
                           {
                                         
                                stage ('Checkout') {
                                        echo 'Checking out SCM'
                                        checkout scm

                                        sh '''
                                                echo "PATH = ${PATH}"
                                                echo "M2_HOME = ${M2_HOME}"
                                        '''
                                }
                    
                                
                                stage ('Docker Build') {
                                        retry(3) {
                                                sh 'docker login -u $SECRET_ID -p $SECRET_PASS innovatecloud.dst.ibm.com:8500'
                                        }
                                        echo 'Building image.'
                                                 sh 'docker build --no-cache -t innovatecloud.dst.ibm.com:8500/nextgen-tooling/dotnet:dotnet_latest .'
                                                 
                                        
                                        echo 'Pushing image tagged :${env.BUILD_ID}.'
                                        retry(3) {
                                                sh 'docker push innovatecloud.dst.ibm.com:8500/nextgen-tooling/dotnet:dotnet_latest'
                                        }
                                        echo 'Pushing image tagged latest.'
                                }

                                stage ('Kubernetes deploy') {
                                        echo 'Setting build No. for k8s deployment.'
                                        echo 'Deploying service.'
                                        sh 'kubectl config set-cluster cluster.local --server=https://innovatecloud.dst.ibm.com:8001 --insecure-skip-tls-verify=true'

                                        sh 'kubectl config set-context cluster.local-context --cluster=cluster.local'

                                        sh 'export icp_username=$SECRET_ID; export icp_password=$SECRET_PASS; section=$(curl -k -X POST -H "Content-Type: application/x-www-form-urlencoded;charset=UTF-8" -d "grant_type=password&username=$icp_username&password=$icp_password&scope=openid" https://innovatecloud.dst.ibm.com:8443/idprovider/v1/auth/identitytoken| sed \'s/[{}"]//g\');arr=($(echo $section | tr "," "\n")); icp_token=$(for i in ${arr[@]}; do echo $i; done| sed -n "/id_token/s/id_token://p");kubectl config set-credentials ntooling@in.ibm.com --token=$icp_token'

                                        sh 'kubectl config set-context cluster.local-context --user=ntooling@in.ibm.com --namespace=nextgen-tooling'
                                        sh 'kubectl config use-context cluster.local-context && kubectl delete svc/dotnet deploy/dotnet pvc/p1-dotnet-1 --ignore-not-found=true && sleep 20 && helm init --tiller-namespace nextgen-tooling && helm install ./helm_icp --tiller-namespace nextgen-tooling'
                        }
                }
            }
        }
    }
}
}
