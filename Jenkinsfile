pipeline {
    agent any
    tools {
        maven 'MVN_HOME'
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "54.234.57.212:8081"
        NEXUS_REPOSITORY = "simplecustomerapp"
        NEXUS_CREDENTIAL_ID = "Nexus-server"
        SCANNER_HOME = tool 'sonar_scanner'
        // Slack details (already configured in Jenkins → Configure System → Slack)
        SLACK_CHANNEL = "#jenkins-integration"
    }
    stages {
        stage("clone code") {
            steps {
                git 'https://github.com/ALEEMUDDIN138/sabear_simplecutomerapp.git'
            }
        }
        stage('Build') {
    steps {
        sh 'mvn -Dmaven.test.failure.ignore=true clean install'
    }
}


        stage("SonarCloud") {
            steps {
                withSonarQubeEnv('sonarqube_server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=Ncodeit \
                        -Dsonar.projectName=Ncodeit \
                        -Dsonar.projectVersion=2.0 \
                        -Dsonar.sources=/var/lib/jenkins/workspace/$JOB_NAME/src/ \
                        -Dsonar.binaries=target/classes/com/visualpathit/account/controller/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports \
                        -Dsonar.jacoco.reportPath=target/jacoco.exec \
                        -Dsonar.java.binaries=src/com/room/sample '''
                }
            }
        }
       stage("publish to nexus") {
    steps {
        script {
            def pom = readMavenPom file: "pom.xml"
            def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
            echo "${filesByGlob[0].name} ${filesByGlob[0].path}"
            def artifactPath = filesByGlob[0].path
            def artifactExists = fileExists artifactPath

            if (artifactExists) {
                nexusArtifactUploader(
                    nexusVersion: NEXUS_VERSION,
                    protocol: NEXUS_PROTOCOL,
                    nexusUrl: NEXUS_URL,
                    groupId: pom.groupId,
                    version: pom.version,
                    repository: NEXUS_REPOSITORY,
                    credentialsId: NEXUS_CREDENTIAL_ID,
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
  //         withCredentials([usernamePassword(credentialsId: 'deployer/******', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
  //             script {
  //                 // Find the WAR file built by Maven
  //                 def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()
  //                 def warName = sh(script: "basename ${warFile} .war | tr '[:upper:]' '[:lower:]'", returnStdout: true).trim()
  //                 echo "Deploying ${warFile} to Tomcat at context path /${warName}..."
  //                 sh """
  //                     curl -u $TOMCAT_USER:$TOMCAT_PASS \\
  //                          -T ${warFile} \\
  //                          "http://52.207.241.86:8080/manager/text/deploy?path=/${warName}&update=true"
        stage("Deploy to Tomcat") {
    steps {
        withCredentials([usernamePassword(credentialsId: 'deployer/******', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
            script {
                // Find the WAR file built by Maven
                def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()
                echo "Deploying ${warFile} to Tomcat at context path /simplecustomerapp ..."
                sh """
                    curl -u $TOMCAT_USER:$TOMCAT_PASS \
                         -T ${warFile} \
                         "http://52.207.241.86:8080/manager/text/deploy?path=/simplecustomerapp&update=true"
                        """
                    }
                }
            }
        }
        stage("Slack Notification") {
            steps {
                slackSend(
                    channel: "${SLACK_CHANNEL}",
                    color: "#36A64F",
                    message: "Declarative pipeline for *Simple Customer App* has been successfully deployed in Tomcat :white_check_mark: by ALEEM for Job: ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
                )
            }
        }
    }
}
