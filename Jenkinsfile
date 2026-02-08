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
             script {
                  bat './mvnw javadoc:javadoc'

                  bat 'if exist doc rmdir /S /Q doc'
                  bat 'mkdir doc'

                  bat 'xcopy /E /I /Y target\\site doc'

                  bat 'powershell -Command "Compress-Archive -Path doc\\* -DestinationPath doc.zip"'

                  archiveArtifacts artifacts: 'doc.zip', fingerprint: true
             }
        }

    }
}
