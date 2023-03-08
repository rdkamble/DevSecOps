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
      
      //stage ('Deploy to Server Application') {
            //steps {
           //sshagent(['server-application']) {
              //sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/vulnweb/target/webgoat-server-v8.2.0-SNAPSHOT.jar ubuntu@13.126.55.0:~/WebGoat'
             //sh 'ssh -o  StrictHostKeyChecking=no ubuntu@13.126.55.0 "nohup java -jar webgoat-server-v8.2.0-SNAPSHOT.jar --server.address=0.0.0.0 --server.port=8080 &"'
    
           //}
          // }     
        //}
      stage ('Deploy to Server Application') {
            steps {
                sshagent(['server-application']) {
                    sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/vulnweb/target/webgoat-server-v8.2.0-SNAPSHOT.jar ubuntu@3.110.49.177:~/WebGoat'
                    sh 'ssh -o  StrictHostKeyChecking=no ubuntu@3.110.49.177 "nohup java -jar webgoat-server-v8.2.0-SNAPSHOT.jar --server.address=0.0.0.0 --server.port=8080 &"'
                }
            }     
       }
      
      
       stage ('Dynamic analysis') {
            steps {
          	    sshagent(['application_server']) {
                    sh 'ssh -o  StrictHostKeyChecking=no ubuntu@43.205.207.201 "sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://3.110.49.177:8080/WebGoat -x zap_report || true" '
			        sh 'ssh -o  StrictHostKeyChecking=no ubuntu@43.205.207.201 "sudo ./zap_report.sh"'
                }      
            }       
        }
      
   }
}  
