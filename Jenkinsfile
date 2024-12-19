pipeline {
    agent any
    stages {
        stage ('SCM checkout') {
            steps {
                script{
                     git url: 'https://github.com/pavangit-lab/ecs-proj.git'
                }
            }
        }
        stage ('SonarQube-SAST'){
            steps {
                script{
                    def scannerHome = tool 'sonarscanner4';
                    withSonarQubeEnv('sonar-pro') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=rocket-nodejs"
                    }
                }
            }
        }
        stage ('Build') {
            steps {
                script{
                     sh 'npm install'
                }
            }
        }
        stage('Docker Build Images') {
            steps {
                script {
                    sh 'docker build -t vign1323/helm-rockets:v1 .'
                    sh 'docker images'
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerPass', variable: 'dockerPassword')]) {
                        sh "docker login -u vign1323 -p ${dockerPassword}"
                        sh 'docker push vign1323/helm-rockets:v1'
                    }
                }
            }
        }
        stage('Trivy Docker-scan') {
            steps {
                script {
                    sh 'trivy image dockerhub1010/helm-rockets:v1 > /home/ubuntu/scan.txt' 
                }
            }
        }
        stage('Deploy on k8s') {
            steps {
                script {
                    withKubeCredentials(kubectlCredentials: [[ credentialsId: 'kubernetes' ]]) {
                        sh 'kubectl create secret generic helm --from-file=.dockerconfigjson=/opt/dockerconfig/config.json  --type kubernetes.io/dockerconfigjson --dry-run=client -oyaml > secret.yaml'
                        sh 'kubectl apply -f secret.yaml'
                        sh 'helm package ./Helm'
                        sh 'helm install myrocket ./myrocketapp-0.1.0.tgz'
                        sh 'helm ls'
                        sh 'kubectl get pods -o wide'
                        sh 'kubectl get svc'
                    }
                }
            }
        }
        stage('Dast Scanning OWASP') {
            steps {
                script{
                    sh "cd ZAP_2.14.0 && ./zap.sh -port 9090 -cmd -quickurl http://18.208.212.61:30008 -quickprogress -quickout ../zap_updated_report.html"
                }
            }
        }
        
    }
}
