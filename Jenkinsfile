pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    enviornment {
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
                sh 'mvn -s setting.xml -DskipTests install'
            }

        }
    }
}