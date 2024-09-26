pipeline {
    agent {
        node {
            label 'jenkins-agent'
        }
    }
    tools {
        maven 'maven-tool'
    }
    stages {
        stage('Build') {
            steps {
                echo "----------Build started--------------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "----------Build completed--------------"
            }
        }
        stage('Unit tests'){
            steps {
                echo "----------Unit Test started--------------"
                sh 'mvn surefire-report:report'
                echo "----------Unit Test completed--------------"
            }
        }
        stage('SonarQube Analysis'){
            environment {
                scannerHome = tool 'sonar'
            }
            steps {
                withSonarQubeEnv('sonar'){
                    sh '${scannerHome}/bin/sonar-scanner'
                }
            }
        }
    }
}