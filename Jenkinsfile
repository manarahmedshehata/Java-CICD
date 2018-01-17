pipeline {
    agent any
    stages { 
      stage('Java Build') {
        steps {
          echo "java build" 
       }
      }
    stage('docker Build') {
      steps { 
        echo "docker build" 
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
            
        
//CCD request
          def cddresponse =httpRequest acceptType: 'APPLICATION_JSON',
           authentication: '123456', contentType: 'APPLICATION_JSON',
            httpMode: 'POST',
            requestBody: '{"applicationName":"innovation_lab_app","applicationVersionName":"1.1","applicationVersionBuildNumber":"'+ build_no+'"} ', 
            responseHandle: 'NONE',
            url: 'http://deploymentcoe.vodafone.skytapdns.com:6063/cdd/design/0000/v1/applications/application-versions/application-version-builds'
          println "Sent a notification, got a $cddresponse response"
          }  
          
        }
      
      }  
    stage('Deployment') {
        steps {
          
          echo "Deployment" 
          
        }
      
      }
    }

}
