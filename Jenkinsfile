pipeline {
    // Jenkins agent yang sudah Anda bangun dengan Docker CLI akan menjalankan pipeline ini.
    // Agent 'any' akan menggunakan node Jenkins yang tersedia.
    agent any

    stages {
        // Tahap 'Checkout SCM' otomatis sudah dilakukan oleh Jenkins di awal pipeline.
        // Tidak perlu stage 'Clone Repository' terpisah di sini.

        stage('Build and Deploy PHP App') {
            steps {
                script {
                    echo "Memulai tahap Build dan Deploy Aplikasi PHP..."

                    // Membuat Dockerfile sementara untuk aplikasi PHP Anda.
                    // Ini akan menggunakan file 'index.php' yang sudah di-clone Jenkins ke workspace.
                    def dockerfileContent = """
                        FROM php:8.2-apache
                        WORKDIR /var/www/html
                        COPY . /var/www/html/
                        EXPOSE 80
                    """
                    writeFile file: 'Dockerfile.app', text: dockerfileContent // Gunakan nama file yang berbeda untuk menghindari konflik dengan Dockerfile Jenkins

                    // Memberi nama image dengan nomor build agar unik
                    def appImageName = "php-simple-app:${env.BUILD_NUMBER}"

                    // Membangun Docker image lokal menggunakan Docker daemon dari agent Jenkins.
                    // Pastikan titik '.' di akhir untuk konteks build adalah direktori workspace Jenkins.
                    sh "docker build -t ${appImageName} -f Dockerfile.app ."
                    echo "Image Docker '${appImageName}' berhasil dibangun."

                    // Menentukan nama container agar unik berdasarkan nomor build
                    def appContainerName = "my-php-app-${env.BUILD_NUMBER}"

                    // Menghentikan dan menghapus container yang mungkin sudah berjalan dari build sebelumnya
                    // Gunakan nama container yang generik atau cari berdasarkan image jika ingin lebih robust
                    sh "docker stop my-minimal-php-app || true" // Stop container dengan nama sebelumnya
                    sh "docker rm my-minimal-php-app || true"   // Remove container dengan nama sebelumnya

                    // Menjalankan Docker container dari image yang baru dibuat
                    // Map port 8082 di host ke port 80 di container
                    sh "docker run -d -p 8082:80 --name ${appContainerName} ${appImageName}"
                    echo "Aplikasi PHP minimal telah di-deploy di http://localhost:8082 dengan container '${appContainerName}'"
                    echo "Anda bisa mengakses aplikasi di: http://localhost:8082"
                }
            }
        }
    }

    post {
        always {
            script {
                echo 'Memulai pembersihan resource Docker...'
                // Membersihkan Docker container setelah pipeline selesai (opsional tapi sangat direkomendasikan)
                // Cari container yang dimulai oleh pipeline ini atau yang menggunakan image ini.
                // Anda bisa menghentikan semua container dengan nama 'my-php-app-*'
                sh 'docker ps -a -q --filter "name=my-php-app-" | xargs -r docker stop || true'
                sh 'docker ps -a -q --filter "name=my-php-app-" | xargs -r docker rm || true'

                // Hapus image yang dibangun oleh pipeline ini
                sh 'docker images -q --filter "reference=php-simple-app:*" | xargs -r docker rmi || true'
                
                echo 'Pembersihan resource Docker selesai.'
                echo 'Pipeline finished.'
            }
        }
    }
}