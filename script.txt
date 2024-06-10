pipeline{
    agent any
    tools{
        jdk 'jdk11'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages{
        stage('Git checkout'){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/ramakrishnayelduti/Ekart.git'
            }
        }
        stage('Compile'){
            steps{
                sh "mvn clean compile -DskipTests=true"
            }
        }
        stage('OWASAP Scan'){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Sonarqube'){
            steps{
                withSonarQubeEnv('sonar-server'){
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectNmae=Devops-CICD \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Devops-CICD '''
                }
                
            }
        }
        stage('Build'){
            steps{
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage('Docker Build and Push'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'bbb8040a-3d35-4df3-9e1e-67fa7e8af486', toolName: 'docker') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart ramakrishnayelduti/shopping-cart:latest"
                        sh "docker push ramakrishnayelduti/shopping-cart:latest"
                    }
                }
            }
        }
    }
}
