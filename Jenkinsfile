
pipeline {
    agent any

    stages {
        stage('GIT') {
            steps {
                echo 'Recuperation du code depuis GitHub'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                dir('backend') {
                    echo 'Compilation du projet Spring Boot'
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Tests') {
            steps {
                dir('backend') {
                    echo 'Execution des tests unitaires'
                    sh 'mvn test'
                }
            }
        }

        stage('SonarQube') {
            steps {
                dir('backend') {
                    echo 'Analyse du code avec SonarQube'
                    withSonarQubeEnv('SonarQube') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo 'Verification du Quality Gate'
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package') {
            steps {
                dir('backend') {
                    echo 'Creation du fichier JAR'
                    sh 'mvn package -DskipTests'
                }

                archiveArtifacts artifacts: 'backend/target/*.jar',
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
