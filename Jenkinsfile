def scmVars

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
                script {
                    scmVars = checkout scm
                }
                echo "scmVars=${scmVars}"
                gitRemoteConfig('joostvdg', 'maven-demo-lib', 'githubtoken')
                sh '''
                git config --global user.email "jenkins@jenkins.io"
                git config --global user.name "Jenkins"
                '''
                //sh 'env'
            }
        }
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean verify'
                }
            }
        }
        stage('Version Bump') {
            when { branch 'master' }
            environment {
                NEW_VERSION = gitNextSemverTagMaven('pom.xml')
            }
            steps {
                container('maven') {
                    sh "mvn versions:set -DnewVersion=${NEW_VERSION}"
                }
                sh 'env'
                gitTag("v${NEW_VERSION}")
                // TODO: change this to env.BRANCH_NAME
            }
        }
        stage('Publish Artifact') {
            when { branch 'master' }
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
    post {
        always {
            cleanWs()
        }
    }
}
