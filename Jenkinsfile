pipeline {
    agent {
        docker { image 'php:8.2-apache' } // Menggunakan Docker image PHP dengan Apache
    }
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Gibransyah/test.git' // Ganti dengan URL repo Anda
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

                    // Membangun Docker image lokal
                    sh "docker build -t php-simple-app-minimal:latest ."
                    
                    // Menjalankan Docker container dari image yang baru dibuat
                    sh "docker run -d -p 8082:80 --name my-minimal-php-app php-simple-app-minimal:latest"
                    echo "Aplikasi PHP minimal telah di-deploy di http://localhost:8082"
                }
            }
        }
    }
    post {
        always {
            // Membersihkan Docker container setelah pipeline selesai (opsional)
            sh 'docker stop my-minimal-php-app || true'
            sh 'docker rm my-minimal-php-app || true'
            sh 'docker rmi php-simple-app-minimal:latest || true'
        }
    }
}