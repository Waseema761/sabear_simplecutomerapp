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

            def pom = readMavenPom file: "pom.xml"
            def files = findFiles(glob: "target/*.${pom.packaging}")

            if (files.length > 0) {

                def artifactPath = files[0].path

                nexusArtifactUploader(
                    nexusVersion: "nexus3",
                    protocol: "http",
                    nexusUrl: "13.59.148.180:8081",
                    groupId: pom.groupId,
                    version: pom.version,
                    repository: "hiring-app",
                    credentialsId: "nexus-creds",
                    artifacts: [
                        [
                            artifactId: pom.artifactId,
                            classifier: '',
                            file: artifactPath,
                            type: pom.packaging
                        ]
                    ]
                )

                echo "Artifact uploaded successfully!"

            } else {
                error "No artifact found in target folder!"
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
