pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests') {
            steps {
                bat 'mvn test'
            }
        }
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://192.168.15.64:9000 -Dsonar.login=bc3ade4b87fa1b77afd490c48fe5b43235ae09fe -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage ('Quality Gate') {
            steps {
                sleep(5) 
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test') {
            steps {
                dir('api-test'){
                    git 'https://github.com/gpsilva/tasks-api-test'
                    bat 'mvn test'
                }
            }
        }
        stage ('Deploy Frontend') {
            steps {
                dir('frontend') {
                    git 'https://github.com/gpsilva/tasks-frontend'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001')], contextPath: 'tasks', war: 'target/tasks.war'
            
                }
            }
        }
        stage ('Functional Test') {
            steps {
                dir('functional-test'){
                    git 'https://github.com/gpsilva/tasks-functional-tests'
                    bat 'mvn test'
                }
            }
        }
        stage('Deploy Prod') {
            steps {
                bat 'docker-compose build'
                bat 'docker-compose up -d'
            }
        }
//        stage ('Health Check') {
//            steps {
//                sleep(20) {
//                    // on interrupt do
//                }
//                dir('functional-test'){
//                    bat 'mvn verify -Dskip.surefire.tests'
//               }
//            }
//       }
   }
   post {
       always {
            junit allowEmptyResults: true, testResults: '/target/surefire-reports/*.xml, api/test/target/surefire-reports/*.xml, functional-test/target/surefire-reports/*.xml, functional-test/target/failsafe-reports/*.xml'
            archiveArtifacts artifacts: 'target/tasks-backend.war, frontend/target/tasks.war', onlyIfSuccessful: true
       }
       unsuccessful {
           emailext attachLog: true, body: 'See attached log' subject: 'Build $BUILD_NUMBER has failed', to: 'gilvan.silva+jenkins@certsys.com.br'
       }
        fixed {
           emailext attachLog: true, body: 'See attached log' subject: 'Build $BUILD_NUMBER has failed', to: 'gilvan.silva+jenkins@certsys.com.br'
       }
    }
}

// https://github.com/gpsilva/tasks-functional-tests
// https://github.com/gpsilva/tasks-api-test
// https://github.com/gpsilva/tasks-frontend
// deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://192.168.0.104:8001')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
// SONAR_LOCAL_QG