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
git branch -m Dev || echo "Branche Dev déjà existante"
git push -u origin Dev || echo "Branche Dev poussée"
'''
            }
        }

        // ------------------------------
        stage('Parallel Builds') {
            parallel {
                stage('Build Dev') {
                    steps {
                        echo "Checkout et build de la branche Dev..."
                        bat '''
git checkout Dev
git pull origin Dev
mvn clean install
'''
                        archiveArtifacts artifacts: 'target/*.jar'
                    }
                }

                stage('Build Main') {
                    steps {
                        echo "Checkout et build de la branche main..."
                        bat '''
git fetch origin main
git checkout main
git pull origin main
mvn clean install
'''
                        archiveArtifacts artifacts: 'target/*.jar'
                    }
                }
            }
        }

        // ------------------------------
        stage('Deploy Dev') {
            steps {
                echo 'Déploiement Docker pour Dev...'
                bat '''
docker-compose down
docker-compose rm -f
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

Les builds Jenkins pour Dev et main ont été effectués ✅
Le déploiement de Dev est terminé ✅

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
