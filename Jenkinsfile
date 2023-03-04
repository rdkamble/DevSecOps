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
            withSonarQubeEnv('sonarqube') {
	            //sh 'mvn sonar:sonar'
                sh 'mvn clean package sonar:sonar'
	        }
	    }
     }
   }
}  
