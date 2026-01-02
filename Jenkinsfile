 pipeline {
    agent any

    tools {
        maven 'MVN_HOME'
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "44.211.69.235:8081"
        NEXUS_REPOSITORY = "pipe-snapshots"
        NEXUS_CREDENTIAL_ID = "Nexus-user"
        SCANNER_HOME = tool 'sonar-scanner'

        // Slack details (already configured in Jenkins → Configure System → Slack)
        SLACK_CHANNEL = "#jenkins-integration"
    }

    stages {
        stage("clone code") {
            steps {
                git 'https://github.com/Waseema761/sabear_simplecutomerapp.git'
            }
        }

        stage("mvn build") {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean install'
            }
        }

    stage('SonarQube') {
    steps {
        withSonarQubeEnv('sonarqube-server') {
            sh '''
            ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
            -Dsonar.projectKey=Ncodeit \
            -Dsonar.projectName=Ncodeit \
            -Dsonar.projectVersion=2.0 \
            -Dsonar.sources=src \
            -Dsonar.java.binaries=target/classes
            '''
        }
    }
}

        stage("publish to nexus") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path}"
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath

                    if (artifactExists) {
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: nexus-creds',
                            artifacts: [
                                [artifactId: pom.artifactId, classifier: '', file: artifactPath, type: pom.packaging],
                                [artifactId: pom.artifactId, classifier: '', file: "pom.xml", type: "pom"]
                            ]
                        )
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }

        // stage("Deploy to Tomcat") {
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'tomcat', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
        //             script {
        //                 // Find the WAR file built by Maven
        //                 def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()
        //                 def warName = sh(script: "basename ${warFile} .war | tr '[:upper:]' '[:lower:]'", returnStdout: true).trim()

        //                 echo "Deploying ${warFile} to Tomcat at context path /${warName}..."

        //                 sh """
        //                     curl -u $TOMCAT_USER:$TOMCAT_PASS \\
        //                          -T ${warFile} \\
        //                          "http://44.199.202.221:8080/manager/text/deploy?path=/${warName}&update=true"

        stage("Deploy to Tomcat") {
    steps {
        withCredentials([usernamePassword(credentialsId: 'tomcat', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
            script {
                // Find the WAR file built by Maven
                def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()

                echo "Deploying ${warFile} to Tomcat at context path /simplecustomerapp ..."

                sh """
                    curl -u $TOMCAT_USER:$TOMCAT_PASS \
                         -T ${warFile} \
                         "http://44.199.202.221:8080//manager/text/deploy?path=/simplecustomerapp&update=true"
        
                        """
                    }
                }
            }
        }

        stage("Slack Notification") {
            steps {
                slackSend(
                    channel: "${SLACK_CHANNEL}",
                    color: "#36a64f",
                    message: "Declarative pipeline for *Simple Customer App* has been successfully deployed in Tomcat ✅ by SNL for Job: ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
                )
            }
        }
    }
}

