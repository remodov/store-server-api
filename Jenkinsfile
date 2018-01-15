#!/usr/bin/env groovy

node {
    stage('checkout') {
        checkout scm
    }

    stage('check java') {
       sh "java -version"
    }

    stage('copy artifact')
    {
       sh "cp /var/lib/jenkins/jobs/store-server-api/builds/lastSuccessfulBuild/archive/build/libs/store-0.0.1-SNAPSHOT.war /home/store-server-api/store.war"
       sh "sudo chown jenkins:jenkins /home/store-server-api/store.war"
       sh "sudo chmod 500 /home/store-server-api/store.war"
    }

     stage('clean') {
       sh "chmod +x gradlew"
       sh "./gradlew clean --no-daemon"
     }

     stage('install tools') {
        sh "./gradlew yarn_install -PnodeInstall --no-daemon"
     }

     stage('backend tests') {
       try {
          sh "./gradlew test -PnodeInstall --no-daemon"
       }
       catch(err) {
         throw err
        }
        finally {
            junit '**/build/**/TEST-*.xml'
        }
     }

     stage('frontend tests') {
        try {
           sh "./gradlew yarn_test -PnodeInstall --no-daemon"
        } catch(err) {
           throw err
        } finally {
           junit '**/build/test-results/karma/TESTS-*.xml'
        }
     }

     stage('packaging') {
        sh "./gradlew bootRepackage -x test -Pprod -PnodeInstall --no-daemon"
        archiveArtifacts artifacts: '**/build/libs/*.war', fingerprint: true
     }

     stage('quality analysis') {
            sh "./gradlew sonarqube --no-daemon"
     }


}
