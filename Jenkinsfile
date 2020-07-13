pipeline{
    agent { label 'agent'
      }
      environment {
          PATH = "opt/apache-maven-3.6.3/bin/mvn"
      }
      stages {
         stage('Open App Server SG') {
            steps {
               withAWS(credentials: 'AWSCred', region: 'ap-south-1') {
               sh 'aws ec2 authorize-security-group-ingress --group-id sg-0afd7f6fc870e581c --protocol tcp --port 22 --source-group sg-0c6cfdd5fcd57aea6'
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
      sh "${PATH} clean package"
      }
    }
        stage('Close App Server SG') {
            steps {
               withAWS(credentials: 'AWSCred', region: 'ap-south-1') {
               sh 'aws ec2 revoke-security-group-ingress --group-id sg-0afd7f6fc870e581c --protocol tcp --port 22 --source-group sg-0c6cfdd5fcd57aea6'
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
