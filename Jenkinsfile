pipeline {

   agent {
       label 'Slave'
   }
  
    environment {
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
        ANSIBLE = tool name: 'Ansible', type: 'com.cloudbees.jenkins.plugins.customtools.CustomTool'
    }
  
   tools {
       // Install the Maven version configured as "M3" and add it to the path.
        maven "M3"
        terraform 'Terraform'
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
                 sh "mvn package -Pdocker -Dmaven.test.skip=true"
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
        stage('Run terraform') {
            steps {
                dir('infrastructure/terraform') { 
                //sh 'terraform init && terraform apply -auto-approve'
                sh 'terraform init && terraform apply -var-file ./panda.tfvars -auto-approve'
                } 
            }
        }
        stage('Copy Ansible role') {
            steps {
                sh 'cp -r infrastructure/ansible/panda/ /etc/ansible/roles/'
            }
        }
        stage('Run Ansible') {
            steps {
                dir('infrastructure/ansible') { 
                sh 'chmod 600 ../panda_kurs.pem'
                sh 'ansible-playbook -i ./inventory playbook.yml'
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