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
  dependencyCheck additionalArguments: '', odcInstallation: 'owasp-dc'
  dependencyCheckPublisher pattern: ''
  archiveArtifacts allowEmptyArchive: true, artifacts: 'dependency-check-report.xml', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
  sh 'rm -rf dependency-check-report.xml*'
	}
}
