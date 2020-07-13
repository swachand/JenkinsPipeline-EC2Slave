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
               sh 'aws ec2 authorize-security-group-ingress --group-id sg-0afd7f6fc870e581c --protocol tcp --port 22  --cidr 172.31.0.0/16' 
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
        stage ('Deployment to App Server') {
           steps {
               sshagent(['test-server']) {
                //App server sshconnet
               sh 'mv target/*.war target/javaapp.war'
               sh "scp -o StrictHostKeyChecking=no  target/javaapp.war ubuntu@13.234.118.183:/opt/apache-tomcat-9.0.37/webapps"
               }
            }    
        }
        stage('Close App Server SG') {
            steps {
               withAWS(credentials: 'AWSCred', region: 'ap-south-1') {
               sh 'aws ec2 revoke-security-group-ingress --group-id sg-0afd7f6fc870e581c --protocol tcp --port 22  --cidr 172.31.0.0/16'
                   //--source-group sg-0c6cfdd5fcd57aea6
               }
           }
        }
        stage('Email Notification') {
            steps {
               mail bcc: '', body: '''HI Team,

Deployment to App Server was succcessfull.

Thanks
Swachand''', cc: '', from: '', replyTo: '', subject: 'Jenkins Pipeline New Commit', to: 'b4oncloud@gmail.com'
               }
           }
      }
 }
