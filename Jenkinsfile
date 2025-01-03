pipeline{
    agent any
    stages{
        stage('Build backend'){
            steps{
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage('Unit Tests'){
            steps{
                bat 'mvn test'
            }
        }
        stage('Sonar Analysis'){
            environment{
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps{
                withSonarQubeEnv('SONAR_LOCAL'){
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=sqp_1a31c9e2d32e2774c14985527a92fcd36689ca6f -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**/*Application.java,**/RootController.java"
                }
            }   
        }
        stage('Quality Gate'){
            steps{
                sleep(5)
                timeout(time: 1, unit: 'MINUTES'){
                   waitForQualityGate abortPipeline: true, credentialsId: 'TOKEN_SONAR'
                }
            }
        }

        stage('Deploy Back-end'){
            steps{
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }

        stage('API Test'){
            steps{
                dir('api-test') {
                    git branch: 'main', url: 'https://github.com/LuisVanique/tasks-api-test'
                    bat 'mvn test'
                }
            }
        }

        stage('Deploy Front-End'){
            steps{
                dir('frontend'){
                    git 'https://github.com/LuisVanique/tasks-frontend'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }

        stage('Functional Test'){
            steps{
                dir('functional-test') {
                    git branch: 'main', url: 'https://github.com/LuisVanique/task-functional-test'
                    bat 'mvn test'
                }
            }
        }

        stage('Deploy prod'){
            steps{
                bat 'docker-compose build'
                bat 'docker-compose up -d'
            }
        }

        stage('Health Check'){
            steps{
                sleep(5)
                dir('functional-test') {
                    bat 'mvn verify -Dskip.surefire.tests'
                }
            }
        }

        post{
            always{
                junit allowEmptyResults: true, stdioRetention: '', testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, 
                functional-test/target/surefire-reports/*.xml, functional-test/target/failsafe-reports/*.xml'
            }
        }
        
    }
}

