pipeline {
    agent any
    stages { 
      stage('Java Build') {
        steps {
          notifyStarted("Java Build")
          echo "java build"
          sh""" 
            mvn clean package -e org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar -Dsonar.host.url=http://deploymentcoe.vodafone.skytapdns.com:9001 -Dmaven.test.skip=true
            mvn clean deploy -Dmaven.test.skip=true
          """
        }
        post
        {
        success{
          notifySuccessful("Java Build")
          }
            failure{
              notifyFailed("Java Build")
               }
          }

      }
     
    stage('docker Build') {
      steps {
        notifyStarted("Docker Build")
        echo "docker build" 
        withCredentials([usernamePassword(credentialsId: '98a29d6f-4f30-485a-a758-475b5fe03274', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh """
            cd deploy/docker/
            cp ${WORKSPACE}/target/account-1.0-SNAPSHOT.war .
            cp /var/jetty-runner-9.4.7.v20170914.jar .
            docker build -t deploymentcoe.vodafone.skytapdns.com/cicd-demo3 .
            docker login --username $USERNAME --password $PASSWORD https://deploymentcoe.vodafone.skytapdns.com
            docker push deploymentcoe.vodafone.skytapdns.com/cicd-demo3
            docker images
            docker rmi deploymentcoe.vodafone.skytapdns.com/cicd-demo3
          """
        }
            
      }
      post
      {
      success{
        notifySuccessful("Docker Build")
        }
          failure{
            notifyFailed("Docker Build")
             }
        }  
    }

    stage('Create Automic pkg') {
      steps {
        echo "Create Automic pkg"
       
        script {
          def build_no= 'pkg_'+ env.BUILD_NUMBER
          echo "test ${build_no}"
          
          def body = '''{
          "name": "'''+ build_no+'''",
          "status": "Active",

          "application": { "name": "TSS Demo" },
          "custom_type": { "name": "Deployment" },
          "owner": { "name": "100/ARA/ARA" },
          "folder": { "name": "DEMOAPP2_PACKAGES" },

          "dynamic": {
            "/github/Remote_Repo_Name": "JAVA-CICD",
            "/github/Server_URL": "https://github.com/manarahmedshehata/"
          }
        }'''
          def response =httpRequest acceptType: 'APPLICATION_JSON', authentication: 'd8739100-6930-4c10-898f-14ab21dfa885',
          contentType: 'APPLICATION_JSON', httpMode: 'POST',
           requestBody: body, 
           responseHandle: 'NONE',
           url: 'http://deploymentcoe.vodafone.skytapdns.com:6062/ara/api/data/v1/packages'
          println "Sent a notification, got a $response response"

        }
      }
    }


    stage('Deployment') {
        steps {
          echo "test"
          echo "call cdd" 
          notifyStarted("Kubernetes Deployment")
          /*          
          sh """
            
            kubectl delete -f deploy/manifests
            kubectl create -f deploy/manifests
            
           """
           */
          //CCD request
          script {
            def build_no= 'pkg_'+ env.BUILD_NUMBER
            def cddresponse =httpRequest acceptType: 'APPLICATION_JSON',
             authentication: '123456', contentType: 'APPLICATION_JSON',
              httpMode: 'POST',
              requestBody: '{"applicationName":"innovation_lab_app","applicationVersionName":"1.1","applicationVersionBuildNumber":"'+ build_no+'"} ', 
              responseHandle: 'NONE',
              url: 'http://deploymentcoe.vodafone.skytapdns.com:6063/cdd/design/0000/v1/applications/application-versions/application-version-builds'
            println "Sent a notification, got a $cddresponse response"  
          }
        }

        post
        {
          success{
            notifySuccessful("Kubernetes Deployment")
          }
          failure{
            notifyFailed("Kubernetes Deployment")
             }
        }
      
      }
    }

}


//functions

def notifyStarted(stagename) {
  // send to Slack
  slackSend (color: '#FFFF00', message: "STARTED: Job $stagename '[${env.BUILD_NUMBER}]'", channel: 'cicd-demo-3')
}

def notifySuccessful(stagename) {
  // send to Slack
  slackSend (color: '#00FF00', message: "SUCCESSFUL: Job $stagename '[${env.BUILD_NUMBER}]'", channel: 'cicd-demo-3')
 
 }

def notifyFailed(stagename) {
  // send to Slack
  slackSend (color: '#FF0000', message: "FAILED: Job $stagename' [${env.BUILD_NUMBER}]'", channel: 'cicd-demo-3')
  // send mail
  emailext attachLog: true, subject: "Jenkins Job ${env.JOB_NAME} $stagename [${env.BUILD_NUMBER}] failed", to: 'yara.abdellatif1@vodafone.com,manar.hassan1@vodafone.com,ahmed.said-abdallah2@vodafone.com', body: """
Dears,
Kindly be informed that the job $stagename has failed, please find the logs attached to this email.
      
Thanks
Deployment CoE
"""
}

