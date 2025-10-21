node {
    stage('SCM') {
        checkout scm
    }

    stage('TruffleHog Secret Scan') {
        // Chạy TruffleHog, ghi báo cáo JSON, bỏ qua lỗi exit code để phân tích tiếp
        sh ' /opt/venv/bin/trufflehog filesystem --repo_path ./ --json > trufflehog-report.json || true '

        // Lưu artifact JSON
        archiveArtifacts artifacts: 'trufflehog-report.json', allowEmptyArchive: true

        // Đọc và in thông tin secrets ra console
        script {
            def report = readJSON file: 'trufflehog-report.json'

            if (report.size() > 0) {
                echo "====== Secrets detected by TruffleHog ======"
                report.each { secret ->
                    echo "File       : ${secret.path}"
                    echo "Commit     : ${secret.commit}"
                    echo "Branch     : ${secret.branch}"
                    echo "Reason     : ${secret.reason}"
                    echo "Start line : ${secret.start_line}"
                    echo "End line   : ${secret.end_line}"
                    echo "Secret     : ${secret.secret.substring(0,5)}*****"
                    echo "--------------------------------------------"
                }
                error("Build failed due to secrets found!")
            } else {
                echo "No secrets detected by TruffleHog."
            }
        }
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

    stage('Dependency-Check Scan') {
        sh 'mkdir -p dependency-check-reports'

        dependencyCheck(
            odcInstallation: 'owasp-dc',
            additionalArguments: '-s ./ -f HTML -f XML -o dependency-check-reports --disableYarnAudit --disableNodeAudit'
        )

        archiveArtifacts allowEmptyArchive: true, artifacts: 'dependency-check-reports/*.html', fingerprint: true
        archiveArtifacts allowEmptyArchive: true, artifacts: 'dependency-check-reports/*.xml', fingerprint: true

        dependencyCheckPublisher(
            pattern: 'dependency-check-reports/dependency-check-report.xml'
        )
    }
}
