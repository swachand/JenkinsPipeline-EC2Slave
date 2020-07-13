pipeline{
    agent { label 'agent'
      }
      environment {
          mvnCMD = "/opt/apache-maven-3.6.3/bin/mvn"
      }
      stages {
         stage('Open App Server SG') {
            steps {
               withAWS(credentials: 'AWSCred', region: 'ap-south-1') {
               sh 'aws ec2 authorize-security-group-ingress --group-id sg-0afd7f6fc870e581c --protocol tcp --port 22 --cidr 172.31.0.0/16' 
                   //--source-group sg-0c6cfdd5fcd57aea6
               }
               
        }
      }
        stage('Installing packages') {
            steps {
               sh 'sudo apt-get -y update && sudo apt-get install -y docker.io openjdk-11-jdk git python3'
               }
        }
        stage ('SCM Checkout'){
          steps {
       git credentialsId: 'gihubt-cred', url: 'https://github.com/swachand/javaproject.git'
      }
    } 
        stage ('Maven Package'){
      steps {
      sh "${mvnCMD} clean package"
      }
    }
    stage ('Build Docker Image'){
      
      //def datestamp = sh(script: 'date +"%M-%S"', returnStdout: true).trim()
     // def version = "latest-${datestamp}
      steps {
      sh  'docker build -t swach/javaproject:v1.1.0.10 .' 
      }
     }
      stage ('Push Docker Image'){
          steps {
      withCredentials([string(credentialsId: 'hubdockerpwd', variable: 'Hublogin')]) {
    
      sh  "docker login -u swach -p ${Hublogin} docker.io"
        
      sh  'docker push swach/javaproject:v1.1.0.10'
         }
      } 
     }
       stage ('Container Run on App Server') { 
       //def DockerRunCMD = 'docker run -d -p 8080:8080 --name myjavapro swach/javaproject:v1.1.0.2'
         steps {
         sshagent(['test-server']) {
      //App server sshconnet
         sh "ssh -o StrictHostKeyChecking=no ubuntu@13.234.118.183 docker run -d -p 8080:8080 --name myjavapro swach/javaproject:v1.1.0.10"
         }
        }    
      }
        stage('Close App Server SG') {
            steps {
               withAWS(credentials: 'AWSCred', region: 'ap-south-1') {
               sh 'aws ec2 revoke-security-group-ingress --group-id sg-0afd7f6fc870e581c --protocol tcp --port 22 --cidr 172.31.0.0/16'
                   //--source-group sg-0c6cfdd5fcd57aea6'
               }
           }
        }
        stage('Email Notification') {
            steps {
               mail bcc: '', body: '''HI Team,

Please check the new commit execution status.

Thanks
Swachand''', cc: '', from: '', replyTo: '', subject: 'Jenkins Pipeline New Commit', to: 'b4oncloud@gmail.com'
               }
           }
    }
}
