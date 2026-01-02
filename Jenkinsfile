 pipeline {
    agent any

    tools {
        maven 'MVN_HOME'
        jdk 'java17'
    }

    environment {
        // ---------- Sonar ----------
        SONAR_SERVER = 'sonarqube-server'
        SCANNER_HOME = tool 'sonar_scanner'

        // ---------- Nexus ----------
        NEXUS_URL = 'http://3.83.214.6:8081'
        NEXUS_REPO = 'pipe-snapshots'
        NEXUS_CREDS = 'Nexus-server'

        // ---------- Tomcat ----------
        TOMCAT_URL = 'http://3.89.121.33:8080'
        TOMCAT_CREDS = 'tomcat'

        // ---------- Slack ----------
        SLACK_CHANNEL = '#jenkins-integration'
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'feature-1.1',
                    url: 'https://github.com/sunil-th/simplecutomerapp.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=SimpleCustomerApp \
                    -Dsonar.projectName=SimpleCustomerApp \
                    -Dsonar.sources=src \
                    -Dsonar.java.binaries=target
                    """
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${NEXUS_CREDS}",
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh """
                    mvn deploy -DskipTests \
                    -Dnexus.username=$NEXUS_USER \
                    -Dnexus.password=$NEXUS_PASS
                    """
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${TOMCAT_CREDS}",
                    usernameVariable: 'TC_USER',
                    passwordVariable: 'TC_PASS'
                )]) {
                    sh '''
                    WAR_FILE=$(ls target/*.war | head -1)
                    APP_NAME=simplecustomerapp

                    echo "Deploying $WAR_FILE to Tomcat..."

                    curl -u $TC_USER:$TC_PASS \
                    -T $WAR_FILE \
                    "$TOMCAT_URL/manager/text/deploy?path=/$APP_NAME&update=true"
                    '''
                }
            }
        }

        stage('Slack Notification') {
            steps {
                slackSend(
                    channel: "${SLACK_CHANNEL}",
                    color: "good",
                    message: "✅ *Simple Customer App* deployed successfully to Tomcat 🚀\nJob: ${JOB_NAME} #${BUILD_NUMBER}"
                )
            }
        }
    }
}

