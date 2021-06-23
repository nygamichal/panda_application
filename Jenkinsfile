pipeline {

   agent {
       label 'Slave'
   }
  
    environment {
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }
  
   tools {
       // Install the Maven version configured as "M3" and add it to the path.
       maven "M3"
   }

    stages {
         stage('Clear running apps') {
             steps {
                 //clear running app from previous built
                 sh 'docker rm -f pandaapp || true' 
                 //w przypadku pierwszego false to i tak drugi mamy true
                 //czyli pipeline idzie dalej
             }
         }
         stage('Get Code') {
             steps {
                 // Get some code from a GitHub repository
                 
                 checkout scm
                 //git branch: 'pipeline', url: 'https://github.com/nygamichal/panda_application.git'
                 
                 
                 // git credentialsId: 'nazwa credentiali', url: 'nazwa repo'
             }
         }
         
         stage('Build and Junit') {
             steps {
                 // Run Maven on a Unix agent.
                sh "mvn clean install"
             }
         }
         stage('Build Docker image'){
             steps {
                 sh "mvn package -Pdocker"
             }
         }
         stage('Run Docker app') {
             steps {
                 sh "docker run -d -p 0.0.0.0:8080:8080 --name pandaapp -t ${IMAGE}:${VERSION}"
             }
         }
         stage('Test Selenium') {
             steps {
                sh "mvn test -Pselenium"
             }
         }
         stage('Deploy jar to artifactory') {
             steps {
                configFileProvider([configFile(fileId: '9d1ed313-ea70-4fa9-9934-7108c53eca75', variable: 'mavensettings')]) {
                // some block
                //sh "mvn -Dmaven.test.failure.ignore=true clean package"
                sh "mvn -s $mavensettings deploy -Dmaven.test.skip=true -e"
               }
             }

        } 
    }

    post {
        always { 
            sh "docker stop pandaapp"
            deleteDir()
        }
    }
}