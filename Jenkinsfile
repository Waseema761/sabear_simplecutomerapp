pipeline {
    agent any

    tools {
        maven 'MVN_HOME'
    }

    environment {
        NEXUS_VERSION        = "nexus3"
        NEXUS_PROTOCOL       = "http"
        NEXUS_URL            = "44.211.69.235:8081"
        NEXUS_REPOSITORY     = "pipe-snapshots"
        NEXUS_CREDENTIAL_ID  = "nexus-user"

        SCANNER_HOME = tool 'sonar-scanner'
        SLACK_CHANNEL = "#jenkins-integration"
    }

    stages {

        stage("Clone Code") {
            steps {
                git 'https://github.com/Waseema761/sabear_simplecutomerapp.git'
            }
        }

        stage("Maven Build") {
            steps {
                sh 'mvn clean install -Dmaven.test.failure.ignore=true'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=Sabear \
                    -Dsonar.projectName=Sabear \
                    -Dsonar.sources=src \
                    -Dsonar.exclusions=**/*.java
                    """
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def files = findFiles(glob: "target/*.${pom.packaging}")

                    if (files.length == 0) {
                        error "No artifact found in target directory"
                    }

                    def artifactPath = files[0].path
                    echo "Uploading artifact: ${artifactPath}"

                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        groupId: pom.groupId,
                        artifactId: pom.artifactId,
                        version: pom.version,
                        artifacts: [
                            [file: artifactPath, type: pom.packaging]
                        ]
                    )
                }
            }
        }

        stage("Deploy to Tomcat") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'tomcat',
                    usernameVariable: 'TOMCAT_USER',
                    passwordVariable: 'TOMCAT_PASS'
                )]) {

                    script {
                        def warFile = sh(
                            script: "ls target/*.war | head -n 1",
                            returnStdout: true
                        ).trim()

                        echo "Deploying ${warFile} to Tomcat..."

                        sh """
                        curl -u $TOMCAT_USER:$TOMCAT_PASS \
                        -T ${warFile} \
                        "http://44.199.202.221:8080/manager/text/deploy?path=/simplecustomerapp&update=true"
                        """
                    }
                }
            }
        }

        stage("Slack Notification") {
            steps {
                slackSend(
                    channel: SLACK_CHANNEL,
                    color: "#36a64f",
                    message: "✅ *SimpleCustomerApp* deployed successfully\nJob: ${JOB_NAME}\nBuild: ${BUILD_NUMBER}"
                )
            }
        }
    }

    post {
        failure {
            slackSend(
                channel: SLACK_CHANNEL,
                color: "danger",
                message: "❌ Pipeline FAILED\nJob: ${JOB_NAME}\nBuild: ${BUILD_NUMBER}"
            )
        }
    }
}
