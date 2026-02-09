pipeline {
    agent any

    options {

        timestamps()                 // Ajoute l’heure dans les logs
        disableConcurrentBuilds()    // Évite deux builds en parallèle

        timestamps()              // Ajoute l’heure dans les logs
        disableConcurrentBuilds() // Évite deux builds en parallèle

    }

    stages {

        // ------------------------------
        stage('Test') {
            steps {

                // Ne casse pas le build si aucun test n'est trouvé

                junit allowEmptyResults: true,
                      testResults: 'target/surefire-reports/*.xml'
            }
        }

        // ------------------------------
        stage('Documentation') {
            steps {

                // Génération de la Javadoc
                bat 'mvnw.cmd javadoc:javadoc'

                // Nettoyage du dossier doc
                bat 'if exist doc rmdir /S /Q doc'
                bat 'mkdir doc'

                // Copie de la documentation
                bat 'xcopy /E /I /Y target\\site doc'

                // Compression en ZIP
                bat 'powershell -Command "if (Test-Path \\"doc.zip\\") { Remove-Item \\"doc.zip\\" -Force }; Compress-Archive -Path doc\\* -DestinationPath doc.zip"'

                // Archivage du ZIP
                archiveArtifacts artifacts: 'doc.zip', fingerprint: true

                // Publication HTML (Javadoc)

                bat 'mvnw.cmd javadoc:javadoc'

                bat 'if exist doc rmdir /S /Q doc'
                bat 'mkdir doc'

                bat 'xcopy /E /I /Y target\\site doc'

                bat '''
                powershell -Command "
                if (Test-Path 'doc.zip') { Remove-Item 'doc.zip' -Force }
                Compress-Archive -Path doc\\* -DestinationPath doc.zip
                "
                '''

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

        // ------------------------------
        stage('Build') {
            steps {

                // Build Maven
                bat 'mvn clean install'

                // Archivage des JAR

                bat 'mvn clean install'

                archiveArtifacts artifacts: 'target/*.jar'
            }
        }

        // ------------------------------

        stage('Notification') {
            steps {
                // Email simple (plugin Mailer)
                mail(
                    subject: "Build réussi ✔",
                    body: """Bonjour,

Le build Jenkins a réussi ✅
=======
        stage('Deploy') {
            steps {
                echo 'Déploiement de l’application avec Docker Compose...'
                bat '''
                docker-compose down
                docker-compose up --build -d
                '''
            }
        }

        // ------------------------------
        stage('Notification') {
            steps {
                mail(
                    subject: "Build & Déploiement réussis ✔",
                    body: """Bonjour,

Le build Jenkins et le déploiement Docker ont réussi ✅
>>>>>>> dac3afe (Initial commit)

Job : ${env.JOB_NAME}
Build : #${env.BUILD_NUMBER}
Lien : ${env.BUILD_URL}

Cordialement,
Jenkins
""",
                    to: "amaniziad66@gmail.com"
                )
            }
        }
    }
}
