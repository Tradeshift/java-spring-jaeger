// This Jenkinsfile uses the declarative syntax. If you need help, check:
// Overview and structure: https://jenkins.io/doc/book/pipeline/syntax/
// For plugins and steps:  https://jenkins.io/doc/pipeline/steps/
// For Github integration: https://github.com/jenkinsci/pipeline-github-plugin
// For credentials:        https://jenkins.io/doc/book/pipeline/jenkinsfile/#handling-credentials
// For credential IDs:     https://ci.ts.sv/credentials/store/system/domain/_/
// Docker:                 https://jenkins.io/doc/book/pipeline/docker/
// Custom commands:        https://github.com/Tradeshift/jenkinsfile-common/tree/master/vars
// Environment variables:  env.VARIABLE_NAME

pipeline {
    agent {
        label "java8 && docker"
    }

    triggers {
        issueCommentTrigger('^(retest|mvn deploy)$')
    }
    options {
        ansiColor('xterm')
        timestamps()
        timeout(time: 60, unit: 'MINUTES') // Kill the job if it gets stuck for too long
        buildDiscarder(logRotator(numToKeepStr: '50', artifactNumToKeepStr: '50'))
    }

    // For Java people
    tools {
        jdk 'openjdk8-jdk'
        maven 'apache-maven-3.6.0'
    }
    environment {
         P12_PASSWORD = credentials 'client-cert-password'
         MAVEN_OPTS = "-Djavax.net.ssl.keyStore=/var/lib/jenkins/.m2/certs/jenkins.p12 \
                       -Djavax.net.ssl.keyStoreType=pkcs12 \
                       -Djavax.net.ssl.keyStorePassword=$P12_PASSWORD"
     }

    stages {
        stage('Clone') {
            steps {
                checkout scm
            }
        }

        stage('Compile') {
            steps {
               sh 'mvn -B -U -T1C clean verify -DskipTests'
            }
        }

        stage('Test') {
            steps {
               sh 'mvn -B -U -T1C test'
            }
        }

        stage('Deploy - PR') {
            when {
                changeRequest()
                expression { pullRequest.comments.any {it.body == 'mvn deploy'} }
            }
            steps {
                dir('deploy') {
                    githubNotify(status: 'PENDING', context: 'deploy', description: 'Deploying version')
                    checkout scm
                    script {
                        def artifactURL = { pom ->
                            def baseURL = 'https://maven.tradeshift.net/content/repositories/tradeshift-public-snapshots/com/tradeshift'
                            return "${baseURL}/${pom.artifactId}/${pom.version}"
                        }

                        def pom = readMavenPom file: 'pom.xml'
                        // build triggered by comment "mvn deploy"
                        echo("This build was triggered by \"mvn deploy\" comment, artifacts will be published to ${artifactURL(pom)}")

                        sh 'mvn -B -U -T1C -DskipTests=true -Dcheckstyle.skip=true clean deploy'

                        pullRequest.comment("Released snapshot artifacts to the maven repo: ${pom.version}")
                        pullRequest.comments.each {
                            if (it.body == 'mvn deploy') {
                                it.delete()
                            }
                        }
                    }
                    githubNotify(status: 'SUCCESS', context: 'deploy', description: 'Deployed version')
                }
            }
        }

        stage('Sonarqube') {
            steps {
                sonarqube()
            }
        }
    }
}
