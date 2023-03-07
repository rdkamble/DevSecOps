pipeline {
  agent any 
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                echo "PATH = ${PATH}"
                echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
    stage ('Check secrets') {
       steps {
         sh 'trufflehog3 https://github.com/rdkamble/sec.git -f json -o truffelhog_output.json || true'
       }
    }
     stage ('Software Composition Analysis') {
            steps {
                dependencyCheck additionalArguments: ''' 
                    -o "./" 
                    -s "./"
                    -f "ALL" 
                    --prettyPrint''', odcInstallation: 'owasp-dc'

                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
       }
  
     stage ('Host vulnerability assessment') {
        steps {
             sh 'echo "In-Progress"'
        }
     }
    
     stage ('Static Application Security Testing') {
	    steps {
            withSonarQubeEnv('Sonar-scanner') {
	            sh 'mvn sonar:sonar'
                //'mvn clean package sonar:sonar'
	        }
	    }
     }
      
      stage ('Deploy to Server Application') {
            steps {
           sshagent(['server-application']) {
              sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/vulnweb/target/webgoat-server-v8.2.0-SNAPSHOT.jar ubuntu@13.126.55.0:~/WebGoat'
             sh 'ssh -o  StrictHostKeyChecking=no ubuntu@13.126.55.0 "nohup java -jar webgoat-server-v8.2.0-SNAPSHOT.jar --server.address=0.0.0.0 --server.port=8080 &"'
    
           }
           }     
        }
      
      stage ('Dynamic Analysis') {
            steps {
           sshagent(['server-application']) {
               sh 'ssh -o  StrictHostKeyChecking=no ubuntu@43.204.28.189 "sudo ./zap.sh -cmd -quickurl http://13.126.55.0:8080/WebGoat -quickprogress -quickout ~/Aut.xml || true" '
           }      
           }       
    }
      
   }
}  
