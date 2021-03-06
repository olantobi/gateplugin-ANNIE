pipeline {
    agent any
    triggers {
        // rebuild this plugin if gate-top gets rebuilt so we can check we haven't introduced a breaking change
        upstream(upstreamProjects: "gate-top", threshold: hudson.model.Result.SUCCESS)
    }
    tools { 
        maven 'Maven Current' 
        jdk 'JDK1.8' 
    }
    options {
        disableConcurrentBuilds()
    }
    stages {
        stage ('Build') {
            steps {
                sh 'mvn -e clean install' 
            }
            post {
                always {
                    junit 'target/surefire-reports/**/*.xml'
                    jacoco exclusionPattern: '**/gate/gui/**,**/gate/resources/**,'
                    warnings canRunOnFailed: true, canResolveRelativePaths: false, consoleParsers: [[parserName: 'Java Compiler (javac)']], defaultEncoding: 'UTF-8', excludePattern: '**/test/**,,**/gate/jape/parser/**', failedNewAll: '0', unstableNewAll: '0', useStableBuildAsReference: true
                }
            }
        }
        stage('Document') {
            when{
                expression { currentBuild.currentResult != "FAILED" && currentBuild.changeSets != null && currentBuild.changeSets.size() > 0 }
            }
            steps {
                sh 'mvn -e -DskipTests site'
            }
            post {
                always {
                    findbugs canRunOnFailed: true, excludePattern: '**/gate/resources/**', failedNewAll: '0', pattern: '**/findbugsXml.xml', unstableNewAll: '0', useStableBuildAsReference: true
                }
                success {
                    step([$class: 'JavadocArchiver', javadocDir: 'target/site/apidocs', keepAll: false])
                }
            }
        }
        stage('Deploy') {
            when{
                branch 'master'
                expression { currentBuild.currentResult == "SUCCESS" && currentBuild.changeSets != null && currentBuild.changeSets.size() > 0 }
            }
            steps {
                sh 'mvn -e -Dmaven.test.skip=true source:jar javadoc:jar deploy'
            }
        }
    }
            }
