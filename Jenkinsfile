node {
    echo "job name is: ${env.JOB_NAME}"
    echo "node name is: ${env.NODE_NAME}"
    echo "build number is: ${env.BUILD_NUMBER}"
def mavenHome = tool name: "maven3.9.3"
properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), pipelineTriggers([pollSCM('* * * * *')])])
    stage('GIT'){
        git branch: 'development', changelog: false, credentialsId: 'ba021c92-6058-4518-9726-7e1fa11e888c', poll: false, url: 'https://github.com/yrr-devops/maven-web-application.git'
    }
    stage('MAVEN'){
        sh "${mavenHome}/bin/mvn clean package"
    }
    stage('SONARQUBE'){
        sh "${mavenHome}/bin/mvn sonar:sonar"
    }
    stage('NEXUS'){
        sh "${mavenHome}/bin/mvn deploy"
    }
    stage('TOMCAT'){
        sshagent(['df5af88e-c06c-4572-9b57-8545831cdaea']) {
    sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@10.42.1.249:/opt/apache-tomcat-9.0.79/webapps/"
}
    }
}
