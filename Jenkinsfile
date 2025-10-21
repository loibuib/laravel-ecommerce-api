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

stage('Scan Vulnerabilities with Dependency-Check') {
    sh 'mkdir -p dependency-check-reports'  // Ensure output directory exists

    dependencyCheck(
        odcInstallation: 'owasp-dc',
        nvdCredentialsId: 'nvd-api',
        additionalArguments: '--nvdApiDelay 1000 -s . -f ALL -o dependency-check-reports'
    )

    archiveArtifacts allowEmptyArchive: true, artifacts: 'dependency-check-reports/*', fingerprint: true
}

}
