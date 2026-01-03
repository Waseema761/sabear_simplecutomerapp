node {

    def GIT_URL = 'https://github.com/Waseema761/sabear_simplecutomerapp.git'
    def GIT_BRANCH = 'feature1'

    try {

        stage('Git Clone') {
            git branch: GIT_BRANCH, url: GIT_URL
        }

       stage('SonarQube Integration') {

    def scannerHome = tool 'sonar-scanner1'

    withSonarQubeEnv('sonarqube-server1') {
        sh """
        ${scannerHome}/bin/sonar-scanner \
        -Dsonar.projectKey=Sabear \
        -Dsonar.projectName=Sabear \
        -Dsonar.projectVersion=1.0 \
        -Dsonar.sources=src
        """
    }
}


        stage('Maven Compilation') {
            sh 'mvn clean install'
        }

        stage('Nexus Artifactory') {
            nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'http',
                nexusUrl: '13.221.106.196:8081',
                groupId: 'in.javahome',
                version: '0.1',
                repository: 'hiring-app',
                credentialsId: 'nexus-user',
                artifacts: [[
                    artifactId: 'hiring',
                    classifier: '',
                    file: 'target/hiring.war',
                    type: 'war'
                ]]
            )
        }

        stage('Deploy On Tomcat') {
            deploy adapters: [
                tomcat9(
                    credentialsId: 'tomcat_credentials',
                    url: 'http://44.204.98.178:8080'
                )
            ],
            contextPath: 'hiring',
            war: 'target/hiring.war'
        }

        stage('Slack Notification') {
            slackSend(
                channel: '#jenkins-integration',
                color: 'good',
                message: "✅ Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }

    } catch (err) {

        slackSend(
            channel: '#jenkins-integration',
            color: 'danger',
            message: "❌ Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        )

        throw err
    }
}

