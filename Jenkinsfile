pipeline {
    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '5', artifactNumToKeepStr: '5', daysToKeepStr: '5', numToKeepStr: '5')
        durabilityHint 'PERFORMANCE_OPTIMIZED'
        timeout(5)
    }
    libraries {
        lib('joostvdg@master')
    }
    agent {
        kubernetes {
            label 'mypod'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: some-label-value
spec:
  containers:
  - name: maven
    image: maven:3-jdk-8
    command:
    - cat
    tty: true
"""
        }
    }
    stages {
        stage('Test versions') {
            steps {
                container('maven') {
                    sh 'uname -a'
                    sh 'mvn -version'
                }
            }
        }
        stage('Checkout') {
            steps {
                git 'https://github.com/joostvdg/maven-demo-lib.git'
            }
        }
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean verify'
                }
            }
        }
        stage('Maven Deploy') {
            steps {
                container('maven') {
                    // #1 = credentialsId for artifactory
                    // #2 = distributionManagement.id
                    generateMavenSettings('artifactory', 'releases')
                    sh 'mvn deploy -s jenkins-settings.xml'
                }
            }
            post {
                always {
                    cleanMavenSettings()
                }
            }
        }
    }
}
