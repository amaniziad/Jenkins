pipeline {
    agent any

    options {
        timestamps()                 // Ajoute l’heure dans les logs
        disableConcurrentBuilds()    // Évite deux builds en parallèle
    }

    stages {

        // ------------------------------
        stage('Test') {
            steps {
                // Exécute les tests Maven et ne casse pas si aucun test n'est trouvé
                junit allowEmptyResults: true,
                      testResults: 'target/surefire-reports/*.xml'
            }
        }

        // ------------------------------
        stage('Documentation') {
            steps {
                echo "Génération de la Javadoc..."
                bat 'mvnw.cmd javadoc:javadoc'

                echo "Nettoyage du dossier doc..."
                bat 'if exist doc rmdir /S /Q doc'
                bat 'mkdir doc'

                echo "Copie de la documentation..."
                bat 'xcopy /E /I /Y target\\site doc'

                echo "Compression de doc en ZIP..."
                // PowerShell compatible Windows + Jenkins
                bat '''
powershell -Command "
if (Test-Path \\"doc.zip\\") { Remove-Item \\"doc.zip\\" -Force }
Compress-Archive -Path doc\\* -DestinationPath doc.zip
"
'''

                echo "Archivage du ZIP..."
                archiveArtifacts artifacts: 'doc.zip', fingerprint: true

                echo "Publication HTML (Javadoc)..."
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
                echo "Build Maven..."
                bat 'mvn clean install'

                echo "Archivage des JAR..."
                archiveArtifacts artifacts: 'target/*.jar'
            }
        }

        // ------------------------------
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
