pipeline {
    agent any

    stages {


        // ------------------------------
        stage('Test') {
            steps {
                // Publier les résultats des tests JUnit
                junit 'target/surefire-reports/*.xml'
            }
        }

        // ------------------------------
        stage('Documentation') {
            steps {
                script {
                    // 1️⃣ Génération de la Javadoc
                    bat 'mvnw.cmd javadoc:javadoc'

                    // 2️⃣ Supprimer et recréer un dossier "doc" propre
                    bat 'if exist doc rmdir /S /Q doc'
                    bat 'mkdir doc'

                    // 3️⃣ Copier la documentation générée
                    bat 'xcopy /E /I /Y target\site doc'

                    // 4️⃣ Compresser la documentation en ZIP
                    bat 'powershell -Command "if (target\site doc.zip) { Remove-Item doc.zip }; Compress-Archive -Path doc\\* -DestinationPath doc.zip"'

                    // 5️⃣ Archiver le ZIP dans Jenkins
                    archiveArtifacts artifacts: 'doc.zip', fingerprint: true

                    // 6️⃣ Publier le rapport HTML
                    // Ici on utilise le fichier index.html généré par Maven Javadoc
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'doc',        // dossier contenant la doc
                        reportFiles: 'index.html', // fichier HTML principal
                        reportName: 'Javadoc',
                        reportTitles: 'Javadoc Report'
                    ])
                }
            }
        }
// ------------------------------
        stage('Build') {
            steps {
                // Nettoyage et compilation Maven
                bat 'mvn clean install'

                // Archivage des JAR générés
                archiveArtifacts artifacts: 'target/*.jar'
            }
        }



    }
}
