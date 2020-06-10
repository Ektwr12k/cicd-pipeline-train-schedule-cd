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
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sshPublisher(
                        continueOnError: false,
                        failOnError: true,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'StagingServer',
                                sshCredentials [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$PASSWORD"
                                ],
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: '/usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && /usr/bin/systemctl start train-schedule'
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
                input 'Are you accept this step?'
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sshPublisher(
                        continueOnError: false,
                        failOnError: true,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'ProductionServer',
                                sshCredentials [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$PASSWORD"
                                ],
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: '/usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && /usr/bin/systemctl start train-schedule'
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
