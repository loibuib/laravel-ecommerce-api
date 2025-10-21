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
        // Using Dependency-Check plugin with NVD API key credential ID 'nvd-api-key'
        dependencyCheck(
            odcInstallation: 'owasp-dc',            // Your Dependency-Check tool installation name
            nvdCredentialsId: 'nvd-api',     // Credential ID for NVD API key stored in Jenkins
            scanPath: '.',                       // Path to scan, current directory
            format: 'ALL',                      // Generate all reports: HTML, XML, JSON
            outputDirectory: 'dependency-check-reports', // Directory for reports
            additionalArguments: '--disableCvssV4'
        )
        // Archive Dependency-Check reports for review in Jenkins
        archiveArtifacts allowEmptyArchive: true, artifacts: 'dependency-check-reports/*', fingerprint: true
    }
}
