pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node20'
    }
    environment {
        SCANNER_HOME=tool 'sonar-check'
        GIT_REPO_NAME = 'Tetris-manifest'
        GIT_USER_NAME = 'Sibapanda22'
        NEW_IMAGE_NAME = 'sibapanda22/tetrisv2:latest'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Sibapanda22/Tetris-V2.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=TetrisVersion2.0 \
                    -Dsonar.projectKey=TetrisVersion2.0 '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-hub', toolName: 'docker'){
                       sh "docker build -t tetrisv2 ."
                       sh "docker tag tetrisv2 sibapanda22/tetrisv2:latest "
                       sh "docker push sibapanda22/tetrisv2:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image sibapanda22/tetrisv2:latest > trivyimage.txt"
            }
        }
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/Sibapanda22/Tetris-menifest.git'
            }
        }
        stage('Update Deployment File') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'github-pat', variable: 'GITHUB_TOKEN')]) {
                       sh "sed -i 's|image: .*|image: ${NEW_IMAGE_NAME}|' deployment.yml"
                       sh 'git add deployment.yml'
                       sh "git commit -m 'Update deployment image to $NEW_IMAGE_NAME'"
                       sh "git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:master"
                    }
                }
            }
        }
    }
}