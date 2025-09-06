pipeline {
    agent any
    tools {
        maven 'MVN_HOME'
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "54.234.57.212:8081/"
        NEXUS_REPOSITORY = "Hiring-app"
        NEXUS_CREDENTIAL_ID = "admin/****** (Nexus-server)"
        SCANNER_HOME = tool 'sonarqube'
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
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        $SCANNER_HOME/bin/sonarqube \
                        -Dsonar.projectKey=Ncodeit \
                        -Dsonar.projectName=Ncodeit \
                        -Dsonar.projectVersion=2.0 \
                        -Dsonar.sources=/var/lib/jenkins/workspace/$JOB_NAME/src/ \
                        -Dsonar.binaries=target/classes/com/visualpathit/account/controller/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports \
                        -Dsonar.jacoco.reportPath=target/jacoco.exec \
                        -Dsonar.java.binaries=src/com/room/sample
                    '''
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
    } // closes stages
} // closes pipeline
