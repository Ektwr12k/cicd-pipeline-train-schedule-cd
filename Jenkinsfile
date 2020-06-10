pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('DeployToStagingServer') {
            when {
                   branch 'master'
            }
            steps {
                withCredentials([AccessToStagingServer(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVeriable: 'USERPASS')]) {
                   sshPublisher(
                     FailOnError: true,
                     continueOnError: false,
                     publishers: [
                           sshPublisherDesc(
                               configName: 'StagingServer',
                               sshCredentials: [
                                   username: "$USERNAME",
                                   encryptedPassphrase: "$USERPASS"
                               ],
                               transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                    )
                               ]
                           )
                     ]
                   )
                }
            }
        }
        stage('DeployToProductionServer') {
            when {
                   branch 'master'
            }             
            steps {
                input 'Does the staging environment look OK?'
                milestone(1)
                withCredentials([AccessToStagingServer(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVeriable: 'USERPASS')]) {
                   sshPublisher(
                     FailOnError: true,
                     continueOnError: false,
                     publishers: [
                           sshPublisherDesc(
                               configName: 'ProductionServer',
                               sshCredentials: [
                                   username: "$USERNAME",
                                   encryptedPassphrase: "$USERPASS"
                               ],
                               transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                    )
                               ]
                           )
                     ]
                   )
                }
            }
        }
    }
}
