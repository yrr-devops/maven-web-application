node {
    echo "job name is: ${env.JOB_NAME}"
    echo "node name is: ${env.NODE_NAME}"
    echo "build number is: ${env.BUILD_NUMBER}"
def mavenHome = tool name: "maven3.9.3"
properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), pipelineTriggers([pollSCM('* * * * *')])])
    sendSlackNotifications('STARTED')
    try{
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
        stage('Trigger DownStreamJob'){
            build job: 'pipelinescriptwithbuildparams'
        }
    }catch(e){
        currentBuild.result="FAILED"
        throw e
    }
    finally{
        sendSlackNotifications(currentBuild.result)
    }
}



def sendSlackNotifications(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary, channel: 'walmart')
}
