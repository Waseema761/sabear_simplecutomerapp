 node {

    def GIT_URL    = 'https://github.com/Waseema761/sabear_simplecutomerapp.git'
    def GIT_BRANCH = 'feature1'

    try {

        stage('Git Clone') {
            git branch: GIT_BRANCH, url: GIT_URL
        }

        stage('Maven Compilation') {
            def mvnHome = tool 'MVN_HOME'
            sh """
            ${mvnHome}/bin/mvn clean install
            """
        }

        stage('SonarQube Integration') {
            def scannerHome = tool 'sonar_scanner'

            withSonarQubeEnv('sonarqube-server1') {
                sh """
                ${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=Sabear \
                -Dsonar.projectName=Sabear \
                -Dsonar.projectVersion=1.0 \
                -Dsonar.sources=src \
                -Dsonar.java.binaries=target/**/WEB-INF/classes
                """
            }
        }

        stage('Nexus Artifactory') {
            nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'http',
                nexusUrl: '13.221.106.196:8081',
                repository: 'hiring-app',
                credentialsId: 'nexus-user',
                groupId: 'in.javahome',
                version: '8-SNAPSHOT',
                artifacts: [[
                    artifactId: 'SimpleCustomerApp',
                    classifier: '',
                    file: 'target/SimpleCustomerApp-8-SNAPSHOT.war',
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
            contextPath: 'SimpleCustomerApp',
            war: 'target/SimpleCustomerApp-8-SNAPSHOT.war'
        }

        stage('Slack Notification') {
            slackSend(
                channel: '#jenkins-integration',
                color: 'good',
                message: "✅ BUILD SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }

    } catch (err) {

        slackSend(
            channel: '#jenkins-integration',
            color: 'danger',
            message: "❌ BUILD FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        )

        throw err
    }
}

