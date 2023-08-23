    pipeline {
        agent any
        tools {
            maven "MAVEN3"
            jdk "OracleJDK8"
        }
        environment {
            SNAP_REPO = 'vprofile-snapshot'
            RELEASE_REPO = 'vprofile-release'
            NEXUS_USER = 'admin'
            NEXUS_PASS = 'Pinspire@3042'
            CENTRAL_REPO = 'vpro-maven-central'
            NEXUSIP = '54.158.217.189'
            NEXUSPORT = '8081'
            NEXUS_GRP_REPO = 'vpro-maven-group'
            NEXUS_LOGIN = 'nexuslogin'
        }

        stages {
            stage('Build') {
                steps {
                    sh 'mvn -s settings.xml -DskipTests install'
                }
                post {
                    success {
                        echo "Now Archiving.."
                        archiveArtifacts artifacts: '**/*.war'
                    }
                }

            }
            stage('Test') {
                steps {
                    sh 'mvn test'
                }
            }
            stage('Checkstyle Analysis') {
                steps {
                    sh 'mvn -s settings.xml checkstyle:checkstyle'
                }
            }
        }
    }