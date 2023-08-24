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
    }
}
