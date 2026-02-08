pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                bat 'mvn clean install'
                archiveArtifacts artifacts: 'target/*.jar'
            }
        }

        stage('Test') {
            steps {
                junit 'target/surefire-reports/*.xml'
            }
        }

        stage('Documentation') {
            steps {
                script {
                    // Génération de la doc
                    bat 'mvnw.cmd javadoc:javadoc'

                    // Supprimer et recréer le dossier doc
                    bat 'if exist doc rmdir /S /Q doc'
                    bat 'mkdir doc'

                    // Copier la documentation
                    bat 'xcopy /E /I /Y target\\site doc'

                    // Compresser en ZIP
                    bat 'powershell -Command "if (Test-Path doc.zip) { Remove-Item doc.zip }; Compress-Archive -Path doc\\* -DestinationPath doc.zip"'

                    // Archiver le ZIP
                    archiveArtifacts artifacts: 'doc.zip', fingerprint: true
                    publishHTML (target : [allowMissing: false,
                     alwaysLinkToLastBuild: true,
                     keepAll: true,
                     reportDir: 'reports',
                     reportFiles: 'myreport.html',
                     reportName: 'My Reports',
                     reportTitles: 'The Report'])
                }
            }
        }


    }
}
