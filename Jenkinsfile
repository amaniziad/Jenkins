pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        // ------------------------------
        stage('Git Branch') {
            steps {
                echo "Création ou renommage de la branche en Dev..."
                bat '''
REM Vérifie la branche actuelle
git branch

REM Renomme la branche locale en Dev
git branch -m Dev || echo "Branche Dev déjà existante"

REM Supprime l'ancienne branche main sur GitHub si elle existe
git push origin :main || echo "Ancienne branche main supprimée"

REM Pousse la nouvelle branche Dev sur GitHub et configure le suivi
git push -u origin Dev
'''
            }
        }

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
                echo 'Déploiement Docker avec suppression automatique des containers existants...'
                bat '''
REM Arrêt et suppression de tous les containers définis dans docker-compose
docker-compose down
docker-compose rm -f

REM Lancement des services
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
