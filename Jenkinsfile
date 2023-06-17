pipeline {
    agent {
        docker {
            image 'maven:3.9.2'
            args '-p 3000:3000'
        }
    }
    environment {
        APP_IMAGE = 'indrasnjya/simple-java-maven-app'
        SHORT_COMMIT = ''
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo '=== Building simple-java-maven-app Docker Image ==='
                script {
                    def app = docker.build(env.APP_IMAGE)
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo '=== Pushing simple-java-maven-app Docker Image ==='
                script {
                    def GIT_COMMIT_HASH = sh(script: "git log -n 1 --pretty=format:'%H'", returnStdout: true).trim()
                    env.SHORT_COMMIT = GIT_COMMIT_HASH[0..7]
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerHubCredentials') {
                        docker.image(env.APP_IMAGE).push(env.SHORT_COMMIT)
                        docker.image(env.APP_IMAGE).push('latest')
                    }
                }
            }
        }        
        stage('Manual Approval') {
            steps {
                script {
                    def userInput = input(
                        id: 'approvalInput',
                        message: 'Lanjutkan ke tahap Deploy?',
                        parameters: [
                            booleanParam(defaultValue: false, description: 'Pilih opsi untuk melanjutkan atau menghentikan pipeline', name: 'APPROVAL')
                        ]
                    )
                    if (userInput) {
                        echo 'Melanjutkan ke tahap Deploy'
                    } else {
                        error('Pipeline dihentikan oleh pengguna')
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sh './scripts/deploy.sh'
                script {
                    def deployInput = input(
                        id: 'deployInput',
                        message: 'Aplikasi telah berhasil di-deploy. Apakah Anda ingin menjeda eksekusi selama 1 menit?',
                        parameters: [
                            booleanParam(defaultValue: false, description: 'Pilih opsi untuk menjeda atau melanjutkan eksekusi', name: 'PAUSE')
                        ]
                    )
                    if (deployInput) {
                        echo 'Menjeda eksekusi selama 1 menit...'
                        sleep(time: 1, unit: 'MINUTES')
                    }
                }
            }
        }
    }
}
