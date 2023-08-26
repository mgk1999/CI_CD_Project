def COLOR_MAP = [
        'SUCCESS': 'good',
        'FAILURE': 'danger'
]
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
            SONARSERVER = 'sonarserver'
            SONARSCANNER = 'sonarscanner'
            registryCredential =  'ecr:us-east-1:awscreds'
            appRegistry  = '943441234686.dkr.ecr.us-east-1.amazonaws.com/vprofileapp'
            vprofileRegistry = 'https://943441234686.dkr.ecr.us-east-1.amazonaws.com'
            cluster = "vproapp"
            service = "vprostagesvc"
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

            stage('Code Analysis With SonarQube') {
                environment {
                    scannerHome = tool "${SONARSCANNER}"
                }
                steps {
                    withSonarQubeEnv("${SONARSERVER}") {
                        sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                                -Dsonar.projectName=vprofile-repo \
                                -Dsonar.projectVersion=1.0 \
                                -Dsonar.sources=src/ \
                                -Dsonar.java.binaries=target \
                                -Dsonar.junit.reportsPath=target/surefire-reports/ \
                                -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                                -Dsonar.java.checkstyle.reportPaths=target/checkstyleresult.xml'''
                    }
//                    timeout(time: 10, unit: 'MINUTES') {
//                        waitForQualityGate abortPipeline: true
//                    }
                }
            }

            stage('UPLOAD ARTIFACT') {
                steps {
                    nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                            groupId: 'QA',
                            version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                            repository: "${RELEASE_REPO}",
                            credentialsId: "${NEXUS_LOGIN}",
                            artifacts: [
                                    [artifactId: 'vproapp' ,
                                     classifier: '',
                                     file: 'target/vprofile-v2.war',
                                     type: 'war']
                            ]
                    )
                }
            }

            stage('Build App image'){
                steps{
                    script{
                        dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER","./Docker-files/app/multistage/")
                    }
                }
            }

            stage('Upload App Image'){
                steps{
                    script{
                        docker.withRegistry( vprofileRegistry, registryCredential ) {
                            dockerImage.push("$BUILD_NUMBER")
                            dockerImage.push('latest')
                        }
                    }
                }
            }


            stage('Deploy to ECS Staging') {
                steps {
                    withAWS(credentials: 'awscreds', region: 'us-east-1') {
                        sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deploymnet'
                    }
                }
            }



        }
        post{
            always {
                echo 'Slack Notifications'
                slackSend channel: '#devops',
                        color: COLOR_MAP[currentBuild.currentResult],
                        message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
            }
        }
    }