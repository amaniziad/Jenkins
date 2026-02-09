pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        // ------------------------------
        stage('Test') {
            steps {
                junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
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
                bat '''
echo if (Test-Path 'doc.zip') { Remove-Item 'doc.zip' -Force } > compress.ps1
echo Compress-Archive -Path doc\\* -DestinationPath doc.zip >> compress.ps1
powershell -NoProfile -File compress.ps1
del compress.ps1
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
                archiveArtifacts artifacts: 'target/*.jar'
            }
        }

        // ------------------------------
        stage('Deploy') {
            steps {
                echo 'Déploiement Docker avec gestion des containers existants...'
                bat '''
REM Stop et supprime l'ancien container si existant
docker stop spring-boot-app || echo "Pas de container à stopper"
docker rm spring-boot-app || echo "Pas de container à supprimer"

REM Déploiement via docker-compose
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
