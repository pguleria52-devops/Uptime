pipeline{
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node18' 
    }
    environment{
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages{
        
        stage('Checkout from GIT'){
            steps{
               git credentialsId: 'github-token',  branch: 'master', url:'https://github.com/pguleria52-devops/Uptime.git'
            }
        }

        stage('Install Dependencies'){
            steps{
                sh "npm install"
            }
        }

        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=uptime \
                        -Dsonar.projectKey=uptime'''
                }
            }
        }

        stage('Quality Gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }

        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.json"
            }
        }

        stage('Docker Build and Push'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t uptime ."
                        sh "docker tag uptime pguleria/uptime:latest"
                        sh "docker push pguleria/uptime:latest"
                    }
                }
            }
        }

         stage("TRIVY IMAGE SCAN") {
            steps {
                sh "trivy image pguleria/uptime:latest > trivy.json"
            }
        }

        stage("Remove container") {
            steps {
                sh "docker stop chatbot || true"
                sh "docker rm chatbot || true"
            }
        }

        stage('Deploy to container') {
            steps {
                sh 'docker run -d --name chatbot -v /var/run/docker.sock:/var/run/docker.sock -p 3001:3001 pguleria/uptime:latest'
            }
        }
    }
}
