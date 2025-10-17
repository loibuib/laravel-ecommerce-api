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
    // Tạo SBOM bằng Syft và lưu lại artifact
    sh 'syft dir:. --output cyclonedx-json=sbom.json'
    archiveArtifacts allowEmptyArchive: true, artifacts: 'sbom*', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
    sh 'rm -rf sbom*'
  }
}
