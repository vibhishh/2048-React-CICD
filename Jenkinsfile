pipeline{
    agent any
    tools{
        jdk 'java17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('Clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'master', url: 'https://github.com/vibhishh/2048-React-CICD.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=2048-Game \
                    -Dsonar.projectKey=2048-Game '''
                }
            }
        }
        stage("Quality Gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Trivy FileSystem Scan') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
       stage('OWASP FileSystem Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t 2048 ."
                       sh "docker tag 2048 mradulsingh25/2048:latest "
                       sh "docker push mradulsingh25/2048:latest "
                    }
                }
            }
        }
        stage("Trivy Image Scan"){
            steps{
                sh "trivy image mradulsingh25/2048:latest > trivy.txt"
            }
        }
        post {
          always {
             emailext attachLog: true,
                subject: "'${currentBuild.result}'",
                body: "Project: ${env.JOB_NAME}<br/>" +
                      "Build Number: ${env.BUILD_NUMBER}<br/>" +
                      "URL: ${env.BUILD_URL}<br/>",
                to: 'mradulsingh1725@gmail.com',
                attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
            }
        }
        stage('Deploy to Container'){
            steps{
                sh 'docker run -d --name 2048 -p 3000:3000 mradulsingh25/2048:latest'
            }
        }
        stage('Deploy to kubernetes'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                       sh 'kubectl apply -f deployment.yaml'
                    }
                }
            }
        }
    }
}
