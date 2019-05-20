node('master') {
  stage('SCM') {
    checkout poll: false, scm: [$class: 'GitSCM', branches: [[name: 'refs/heads/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/uladzimiramelyanovich/curl.git']]]
  }
  stage ('Groovy samples') {
	x = 1
	println x
	x = new java.util.Date()
	println x
	x = -3.1499392
	println x
	x = false
	println x
	x = "Hi"
	println x
  }
  stage('SonarQube Analysis') {

  sh "/var/jenkins_home/tools/sonar-scanner-3.3.0.1492-linux/bin/sonar-scanner -Dsonar.host.url=http://mysonar:9000 -Dsonar.projectName=curl -Dsonar.projectVersion=1.0 -Dsonar.projectKey=curl:app -Dsonar.sources=curl/src -Dsonar.projectBaseDir=/var/jenkins_home/workspace/"
    }
  }
