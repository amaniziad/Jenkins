pipeline {
    agent any

    stages {

        stage('Test') {
            steps {
                junit 'target/surefire-reports/*.xml'
            }
        }

        stage('Documentation') {
            steps {
                bat 'mvnw.cmd javadoc:javadoc'
                bat 'if exist doc rmdir /S /Q doc'
                bat 'mkdir doc'
                bat 'xcopy /E /I /Y target\\site doc'
                bat 'powershell -Command "if (Test-Path \\"doc.zip\\") { Remove-Item \\"doc.zip\\" }; Compress-Archive -Path doc\\* -DestinationPath doc.zip"'
                archiveArtifacts artifacts: 'doc.zip', fingerprint: true

                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'doc',
                    reportFiles: 'index.html',
                    reportName: 'Javadoc'
                ])
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean install'
                archiveArtifacts artifacts: 'target/*.jar'
            }
        }

        // ✅ Email avec steps
        stage('Notification') {
            steps {
                mail(
                    subject: "Build Réussi ",
                    body: "Le build a réussi",
                    to: "amaniziad66@gmail.com",
                    from: "amaniziad66@gmail.com"
                )
            }
        }
    }
}
