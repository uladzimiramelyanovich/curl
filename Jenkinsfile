node('agent') {
  stage('SCM') {
    checkout poll: false, scm: [$class: 'GitSCM', branches: [[name: 'refs/heads/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/uladzimiramelyanovich/curl.git']]]
  }
  stage('Build') {
    sh "./buildconf"
    sh "./configure"
    sh "make"
	
	def server = Artifactory.server 'autoartifactory'
	def uploadSpec = """{
        "files": [
            {
              "pattern": "*.txt",
              "target": "example-repo-local/files/"
            }
        ]
    }"""
	server.upload spec: uploadSpec, failNoOp: true
  }
  stage('Unit Tests') {
    sh "make test"
  }
  stage('SonarQube analysis') {
    // requires SonarQube Scanner 2.8+
    def scannerHome = tool 'Sonarqube Scanner 3.3.0.1492';
    withSonarQubeEnv('sonarqube') {
      sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectName=curl -Dsonar.projectVersion=1.0 -Dsonar.projectKey=curl:app -Dsonar.scm.provider=git -Dsonar.sources=./src"	 
      println 'Sending data to Sonarqube server.'
	  sleep(30)	
    }
  }
  stage('Quality Gate') {
    timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
      def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
      if (qg.status != 'OK') {
        error "Pipeline aborted due to quality gate failure: ${qg.status}"
      }
    }
  }
  stage('Atrifactory') {
    println 'Sending data to Artifactory'
  }  
}	
  
