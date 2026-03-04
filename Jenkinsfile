pipeline {
    agent any

    tools {
        maven "maven"
    }

    environment {
        NEXUS_URL = "13.59.148.180:8081"
        NEXUS_REPOSITORY = "hiring-app"
        NEXUS_CREDENTIAL_ID = "nexus-creds"

        SONAR_HOST_URL = "http://13.59.148.180:9000"
    }

    stages {

        stage("Checkout Code") {
            steps {
                git branch: 'feature-1.1',
                url: 'https://github.com/Waseema761/sabear_simplecutomerapp.git'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=sabear-customer-app \
                    -Dsonar.host.url=${SONAR_HOST_URL}
                    """
                }
            }
        }

        stage("Quality Gate Check") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Publish Artifact to Nexus") {
            steps {
                script {

                    def artifactPath = "target/SimpleCustomerApp.war"

                    if (fileExists(artifactPath)) {

                        nexusArtifactUploader(
                            nexusVersion: "nexus3",
                            protocol: "http",
                            nexusUrl: NEXUS_URL,
                            groupId: "com.javatpoint",
                            version: "${BUILD_NUMBER}",
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [
                                    artifactId: "SimpleCustomerApp",
                                    classifier: '',
                                    file: artifactPath,
                                    type: "war"
                                ]
                            ]
                        )

                        echo "Artifact Uploaded Successfully!"

                    } else {
                        error "WAR file not found!"
                    }
                }
            }
        }

    }

    post {
        success {
            echo "Pipeline Completed Successfully"
        }

        failure {
            echo "Pipeline Failed"
        }
    }
}
