pipeline {
    agent any
    stages {
        stage('build') {
            steps {
                echo "Running build automation and creating zip archive"
                sh './gradlew build --no-daemon'
                archiveArtifacts 'dist/trainSchedule.zip'
            }
        }
        stage('deploy to staging') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                sshPublisher( 
                    failOnError: true,
                    continueOnError: false,
                    publishers: [ 
                        sshPublisherDesc(
                            configName: 'staging',
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

        stage('deploy to production') {
            when {
                branch 'master'
            }
            steps {
                input 'does the stage environment looks good?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', passwordVariable: 'USERPASS', usernameVariable: 'USERNAME')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'production',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ],
                        transfers: [
                            sshTransfer(
                                execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule ', 
                                remoteDirectory: '/tmp',
                                removePrefix: 'dist/',
                                sourceFiles: 'dist/trainSchedule.zip'
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
