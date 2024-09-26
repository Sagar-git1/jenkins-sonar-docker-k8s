def registry = 'https://jenkinsuni.jfrog.io'
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
        stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrogtoken"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "maven-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
            }
        }   
    }
    }
}