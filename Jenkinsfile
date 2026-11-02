
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
                withCredentials([
                usernamePassword(
                    credentialsId: 'mysql-credentials',
                    usernameVariable: 'DB_USERNAME',
                    passwordVariable: 'DB_PASSWORD'
                )
            ]) {
                withEnv([
                    'DB_URL=jdbc:mysql://172.18.0.2:3306/test_db?createDatabaseIfNotExist=true'
                ]) {
                    sh 'mvn test'
                }
            }
        }
    }
}



stage('SonarQube') {
    steps {
        dir('backend') {
            echo 'Analyse du code avec SonarQube'

            withSonarQubeEnv('sq1') {
                sh '''
                    mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                        -Dsonar.projectKey=votre-projet-Spring-Boot
                '''
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
