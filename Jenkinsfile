pipeline {
    agent {
        docker {
            image 'maven:3.9.0'
            args '-v /root/.m2:/root/.m2'
        }
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
            def app = docker.build("indrasnjya/simple-java-maven-app")
            env.APP_IMAGE = "indrasnjya/simple-java-maven-app" // Menyimpan nama image dalam environment variable
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
            def SHORT_COMMIT = GIT_COMMIT_HASH[0..7]
            docker.withRegistry('https://registry.hub.docker.com', 'dockerHubCredentials') {
                docker.image(env.APP_IMAGE).push(SHORT_COMMIT) // Menggunakan env.APP_IMAGE untuk mengakses nama image
                docker.image(env.APP_IMAGE).push('latest') // Menggunakan env.APP_IMAGE untuk mengakses nama image
            }
        }
    }
}
stage('Remove local images') {
    steps {
        echo '=== Delete the local docker images ==='
        sh 'docker rmi -f ${env.APP_IMAGE}:latest || :' // Menggunakan env.APP_IMAGE untuk mengakses nama image
        sh "docker rmi -f ${env.APP_IMAGE}:${SHORT_COMMIT} || :" // Menggunakan env.APP_IMAGE dan SHORT_COMMIT untuk mengakses nama image
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
                sh './script/deploy.sh'
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
