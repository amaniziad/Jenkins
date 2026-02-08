pipeline {
 agent any
 stages {

 stage('build') {
 steps {
 bat 'mvn clean install'
 archiveArtifacts artifacts: 'target/*.jar'
 junit 'target/surefire-reports/*.xml'
}
}
}
}