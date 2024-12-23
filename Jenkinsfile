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
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=sqp_2d3d73ac72493f6e764a5279c67322bcdc8d41a0 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**/*Application.java,**/RootController.java"
                }
            }   
        }
        stage('Quality Gate'){
            steps{
                timeout(time: 1, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
    }
}