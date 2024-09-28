def imageName = 'jenkinsuni.jfrog.io/dockermvn/javamvn'
def registry = 'https://jenkinsuni.jfrog.io/'
pipeline {
    agent {
        node {
            label 'jenkins-agent'
        }
    }
    environment {
            ARTIFACTORY_SERVER = 'jfrogserver'
            ARTIFACTORY_REPO = 'mvn-libs-release-local'
    }
    tools {
        maven 'maven-tool'
        jfrog 'jfrogcli'
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
                scannerHome = tool 'sonarscanner' // sonar scanner is the name given when you installed sonar scanner tool under tools section from manage jenkins
            }
            steps {
                withSonarQubeEnv('sonar'){ //here sonar is the name given when you add sonar server details in system configuration under manage jenkins
                    sh '${scannerHome}/bin/sonar-scanner'
                }
            }
        }
        stage("Quality Gate"){
            steps {
                script {
                timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
                def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
                }
                }
            }
        }
        stage('Extract version'){
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml' //readMavenPOM will be coming from Pipeline Utility Steps plugin 
                    env.VERSION = pom.version
                    echo "Version: ${env.VERSION}"
                }
            }
        }
        stage('Publish to Artifactory') {
            steps {
                jf 'rt u target/*.jar ${ARTIFACTORY_REPO}/${env.VERSION}/ --server-id ${ARTIFACTORY_SERVER}'
            }
        }
        stage('Docker image build'){
            steps {
                script {
                    echo '-------------Docker image build started---------'
                    app = docker.build(imageName":"env.VERSION)
                    echo '-------------Docker image build completed-------'
                    
                }
            }
        }
        stage('Docker image publish'){
            steps {
                script {
                    docker.withRegistry(registry,'jfrog-token'){
                        app.push()
                    }
                }
            }
        }
    }
}