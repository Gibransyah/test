pipeline {
    agent any // Jenkins agent (yang sudah Anda bangun dengan Docker CLI) akan menjalankan pipeline ini

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Gibransyah/test.git' // Pastikan ini adalah URL repo Anda
            }
        }

        stage('Deploy Application using Docker Local Image') {
            steps {
                script {
                    // Membuat Dockerfile sementara untuk aplikasi PHP Anda
                    def dockerfileContent = """
                        FROM php:8.2-apache
                        COPY . /var/www/html/
                        EXPOSE 80
                    """
                    writeFile file: 'Dockerfile', text: dockerfileContent

                    // Membangun Docker image lokal menggunakan Docker daemon dari agent Jenkins
                    sh "docker build -t php-simple-app-minimal:latest ."
                    
                    // Menghentikan dan menghapus container sebelumnya jika ada
                    sh "docker stop my-minimal-php-app || true"
                    sh "docker rm my-minimal-php-app || true"

                    // Menjalankan Docker container dari image yang baru dibuat
                    sh "docker run -d -p 8082:80 --name my-minimal-php-app php-simple-app-minimal:latest"
                    echo "Aplikasi PHP minimal telah di-deploy di http://localhost:8082"
                }
            }
        }
    }
    post {
        always {
            script { // Pastikan 'sh' command di dalam 'script' block
                // Membersihkan Docker container setelah pipeline selesai (opsional tapi direkomendasikan)
                sh 'docker stop my-minimal-php-app || true'
                sh 'docker rm my-minimal-php-app || true'
                sh 'docker rmi php-simple-app-minimal:latest || true' // Menghapus image yang dibangun
                echo 'Pembersihan resource Docker selesai.'
                echo 'Pipeline selesai.'
            }
        }
    }
}