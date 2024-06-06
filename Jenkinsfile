pipeline {
    agent {
		label 'jdk8'
    }
    environment {
        APP_URL = 'http://localhost:8080' // Change this to your application's URL
    }
    stages {
        stage('Build and Verify') {
            steps {
                container('maven') {
                    sh 'mvn clean verify'
                }
            }
        }
        stage('Run ZAP Scan') {
            steps {
                container('zap') {
                    script {
                        // Start ZAP in daemon mode
                        zap.sh -daemon -host 0.0.0.0 -port 8090 -config api.disablekey=true -config gui.enable=false
                        
                        // Wait for ZAP to start
                        sleep(time: 30, unit: 'SECONDS')
                        
                        // Run ZAP spider to crawl the target application
                        sh "zap-cli --zap-url http://localhost:8090 spider ${env.APP_URL}"
                        
                        // Run ZAP active scan
                        sh "zap-cli --zap-url http://localhost:8090 active-scan ${env.APP_URL}"
                        
                        // Wait for the scan to complete
                        sh 'zap-cli --zap-url http://localhost:8090 wait-for-issues -i 120'
                        
                        // Generate report
                        sh 'zap-cli --zap-url http://localhost:8090 report -o zap-report.html -f html'
                        
                        // Archive the report in Jenkins
                        archiveArtifacts artifacts: 'zap-report.html', allowEmptyArchive: true
                    }
                }
            }
        }
    }
}
