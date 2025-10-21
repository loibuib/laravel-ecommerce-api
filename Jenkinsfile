node {
	stage('SCM') {
		checkout scm
	}

	stage('SonarQube Analysis') {
	    def scannerHome = tool 'SonarScanner'
	    withSonarQubeEnv() {
		    sh "${scannerHome}/bin/sonar-scanner"
	    }
	}

	stage('Generate SBOM') {
	    sh 'syft dir:. --output cyclonedx-json=sbom.json'
	    archiveArtifacts allowEmptyArchive: true, artifacts: 'sbom.json', fingerprint: true
	}

	stage('Scan Vulnerabilities with Grype') {
	    sh 'grype sbom:sbom.json --output json > grype-report.json'
	    archiveArtifacts allowEmptyArchive: true, artifacts: 'grype-report.json', fingerprint: true
	}
  
stage('Dependency-Check') {
  steps {
    // Run dependency check scan
    dependencyCheck additionalArguments: '''
      --nvdApiResultsPerPage 2000 
      --data /var/jenkins_home/dependency-check-data 
      --format XML
      --prettyPrint
    ''',
    odcInstallation: 'owasp-dc',
    nvdCredentialsId: 'nvd-api'

    // Publish and archive results
    dependencyCheckPublisher pattern: 'dependency-check-report.xml'

    archiveArtifacts(
      allowEmptyArchive: true,
      artifacts: 'dependency-check-report.xml',
      fingerprint: true,
      followSymlinks: false,
      onlyIfSuccessful: true
    )

    // Clean workspace artifacts
    sh 'rm -f dependency-check-report.xml*'
  }
}

}
