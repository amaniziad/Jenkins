pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Git Branch') {
            steps {
                echo "Création ou renommage de la branche en Dev..."
                bat '''
git branch -m Dev || echo "Branche Dev déjà existante"
git push -u origin Dev || echo "Branche Dev poussée"
'''
            }
        }

        stage('Tests & Documentation') {
            parallel(
                "Unit Testing": {
                    steps {
                        echo "Exécution des tests unitaires..."
                        bat 'mvn test'
                        junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                    }
                },
                "Documentation": {
                    steps {
                        echo "Génération de la Javadoc..."
                        bat 'mvn javadoc:javadoc'

                        bat 'if exist doc rmdir /S /Q doc'
                        bat 'mkdir doc'
                        bat 'xcopy /E /I /Y target\\site doc'

                        bat '''
echo if (Test-Path 'doc.zip') { Remove-Item 'doc.zip' -Force } > compress.ps1
echo Compress-Archive -Path doc\\* -DestinationPath doc.zip >> compress.ps1
powershell -NoProfile -File compress.ps1
del compress.ps1
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
            )
        }

        stage('Build') {
            steps {
                echo "Build Maven..."
                bat 'mvn clean install'
                archiveArtifacts artifacts: 'target/*.jar'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Déploiement Docker...'
                bat '''
docker-compose down
docker-compose rm -f
docker-compose up --build -d
'''
            }
        }

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
