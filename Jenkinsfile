
pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    stages {

        stage('GIT') {
               steps {
                echo 'Checkout du projet depuis GitHub'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Compilation du projet'
                sh 'mvn clean compile'
            }
        }

        stage('Tests') {
            steps {
                echo 'Execution des tests unitaires'
                sh 'mvn test'
            }
        }

        stage('SonarQube') {
            steps {
                echo 'Analyse SonarQube'

                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo 'Verification Quality Gate'

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package') {
            steps {
                echo 'Creation du fichier JAR'

                sh 'mvn package -DskipTests'

                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'Pipeline termine avec succes !'
        }

        failure {
            echo 'Echec du pipeline !'
        }
    }
}
