pipeline {
 agent any
 stages {

stage('Test') {
 steps {
 junit 'target/surefire-reports/*.xml'


 stage('build') {
 steps {
 bat 'mvn clean install'
 archiveArtifacts artifacts: 'target/*.jar'
}
}
}
}
}
}