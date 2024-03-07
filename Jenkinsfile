pipeline {
    agent any
    
    stages {
        stage('Checkout project') {
            steps {
                echo 'Downloading git directory...'
                git 'https://github.com/futehc/tustsxvwa.git'
            }
        }
        
        stage('Retire.js Scan') {
            steps {
                script {
                    echo 'Running Retire.js scan...'
                    def retireJsOutputFile = "${WORKSPACE}/RetireJs-${BUILD_NUMBER}.json"
                    def retireJsScriptPath = "C:/Users/bilal.abid/AppData/Roaming/npm/retire.cmd"

                    // Run Retire.js scan on the root directory
                    def scanExitCode = bat returnStatus: true, script: "\"${retireJsScriptPath}\" --path \"${WORKSPACE}\" --outputformat jsonsimple --outputpath \"${retireJsOutputFile}\""
                    
                    if (scanExitCode == 0) {
                        echo 'Retire.js scan completed successfully.'
                    } else {
                        echo 'Retire.js scan failed to run.'
                    }
                }
            }
        }
        
        stage('Analyze Reports') {
            steps {
                script {
                    // Read JSON file using jq
                    def retireJsOutputFile = "${WORKSPACE}/RetireJs-${BUILD_NUMBER}.json"
                    def retireJsReport = bat(script: "jq . ${retireJsOutputFile}", returnStdout: true).trim()
                    evaluateRetireJsReport(retireJsReport)
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}

def evaluateRetireJsReport(retireJsReport) {
    int criticalCount = retireJsReport.count('critical')
    int highCount = retireJsReport.count('high')
    int mediumCount = retireJsReport.count('medium')

    echo "Critical issues found: ${criticalCount}"
    echo "High issues found: ${highCount}"
    echo "Medium issues found: ${mediumCount}"
    
    if (criticalCount >= 1) {
        error 'Retire.js found at least one critical vulnerability. Build failed.'
    } else if (highCount >= 5) {
        error 'Retire.js found at least five high severity vulnerabilities. Build failed.'
    } else if (mediumCount >= 10) {
        error 'Retire.js found at least ten medium severity vulnerabilities. Build failed.'
    } else {
        echo 'No critical, high or medium severity vulnerabilities found.'
    }
}
