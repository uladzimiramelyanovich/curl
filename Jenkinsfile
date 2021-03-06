node('agent') {
    stage('Checkout repository') {
        checkout poll: false, scm: [$class: 'GitSCM', branches: [[name: 'refs/heads/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/uladzimiramelyanovich/curl.git']]]
    }

    stage('Build and execute Unit Tests') {
        sh "./buildconf"
        sh "./configure --enable-debug"
		sh "make"
		sh "cd ./tests"
		sh "make test"
	}
	stage('SonarQube analysis') {
		// requires SonarQube Scanner 2.8+
		def scannerHome = tool 'Sonarqube Scanner 3.3.0.1492';
		withSonarQubeEnv('sonarqube') {
			sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectName=curl -Dsonar.projectVersion=1.0 -Dsonar.projectKey=curl:app -Dsonar.scm.provider=git -Dsonar.sources=. -Dsonar.cxx.includeDirectories=**/include/*"	 
		}
	}
	stage('Build') {
		sh "./buildconf"
		sh "./configure"
		sh "make"
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
		def server = Artifactory.server 'artifactory'
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
}	
  
