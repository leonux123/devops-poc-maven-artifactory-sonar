node {
	stage ('Clone') {
        git url: 'https://github.com/leonux123/devops-poc-maven-artifactory-sonar.git'
}
	stage('SonarQube analysis') { 
        withSonarQubeEnv('My SonarQube Server') { 
          sh 'mvn clean package org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar ' + 
          '-f pom.xml ' +
          '-Dsonar.projectKey=com.leonux123:all:master ' +
          '-Dsonar.login=$SONAR_UN ' +
          '-Dsonar.password=$SONAR_PW ' +
          '-Dsonar.language=java ' +
          '-Dsonar.sources=. ' +
          '-Dsonar.tests=. ' +
          '-Dsonar.test.inclusions=**/*Test*/** ' +
          '-Dsonar.exclusions=**/*Test*/**'
        }
    }
	stage("SonarQube Quality Gate") { 
        timeout(time: 1, unit: 'HOURS') { 
           def qg = waitForQualityGate() 
           if (qg.status != 'OK') {
             error "Pipeline aborted due to quality gate failure: ${qg.status}"
           }
        }
    }
	stage ('Distribute binaries') { 
    def SERVER_ID = 'ART'
    def server = Artifactory.server SERVER_ID
    def uploadSpec = 
    """
    {
    "files": [
        {
            "pattern": "target/*.jar",
            "target": "bazinga-repo/com/leonux123/jar/{1}/"
        }
      ]
    }
    """
    def buildInfo = Artifactory.newBuildInfo() 
    buildInfo.env.capture = true 
    buildInfo=server.upload(uploadSpec) 
    server.publishBuildInfo(buildInfo) 
}
    }
