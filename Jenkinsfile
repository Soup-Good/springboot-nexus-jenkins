pipeline {
    agent any

    options {
        skipStagesAfterUnstable()
    }

    tools {
        maven '3.9.11'
    }

    environment {
        NEXUS_URL = 'http://nexus:8081'
        NEXUS_REPOSITORY = 'springboot-releases'

        GROUP_ID = 'com.example'
        ARTIFACT_ID = 'springboot-helloworld'
        VERSION = "1.0.${BUILD_NUMBER}"
        PACKAGING = 'jar'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Soup-Good/springboot-nexusjenkins.git'
            }
        }

        stage('Verify Tools') {
            steps {
                sh 'git --version'
                sh 'mvn --version'
                sh 'java -version'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('Build and Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Show Generated JAR') {
            steps {
                sh 'ls -lh target/*.jar'
            }
        }

        stage('Archive Artifact in Jenkins') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Upload JAR to Nexus') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-creds',
                        usernameVariable: 'NEXUS_USERNAME',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )
                ]) {
                    sh '''
                        JAR_FILE=$(ls target/*.jar | head -n 1)

                        echo "Uploading Spring Boot JAR artifact to Nexus..."
                        echo "JAR file found: $JAR_FILE"
                        echo "Nexus repository: ${NEXUS_URL}/repository/${NEXUS_REPOSITORY}/"

                        mvn deploy:deploy-file \
                          -DgroupId=${GROUP_ID} \
                          -DartifactId=${ARTIFACT_ID} \
                          -Dversion=${VERSION} \
                          -Dpackaging=${PACKAGING} \
                          -Dfile=$JAR_FILE \
                          -DrepositoryId=nexus \
                          -Durl=${NEXUS_URL}/repository/${NEXUS_REPOSITORY}/ \
                          -DgeneratePom=true \
                          -Dusername=${NEXUS_USERNAME} \
                          -Dpassword=${NEXUS_PASSWORD}
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }

        success {
            echo 'Build successful. The JAR file was created, archived in Jenkins, and uploaded to Nexus.'
        }

        failure {
            echo 'Build failed. Check the Jenkins Console Output for the error.'
        }
    }
}
