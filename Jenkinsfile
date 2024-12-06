pipeline {
    agent any //Menentukan agen eksekusi untuk pipeline ini. Dalam hal ini, pipeline dapat berjalan di agen Jenkins mana saja.
    parameters {
        choice(name: 'BUILD_TYPE', choices: ['Scan Only', 'Scan + Deploy'], description: 'Select build type')
        //Parameter pilihan memungkinkan pengguna memilih jenis build:
        //1. "Scan Only": Melakukan analisis kode dengan SonarQube.
        //2. "Scan + Deploy": Melakukan analisis kode dan juga deploy ke staging.
    }
    environment {
        SONAR_TOKEN = credentials('token_sonar')
        //Variabel lingkungan yang menyimpan token autentikasi SonarQube
        //menggunakan kredensial yang disimpan di Jenkins.
    }
    stages {
        stage('Check Environment') {
            steps {
                sh '''
                if ! dpkg -s python3-venv > /dev/null 2>&1; then
                    echo "python3-venv is not installed. Installing now..."
                    sudo apt update && sudo apt install -y python3-venv
                fi
                '''
                //Mengecek apakah modul `python3-venv` sudah terinstal.
                //Jika belum, modul akan diinstal menggunakan `apt`.
            }
        }
        stage('Install Dependencies') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install -r requirements.txt
                '''
                //Membuat virtual environment Python di direktori `venv`,
                //lalu mengaktifkannya dan menginstal semua dependensi yang
                //tercantum dalam file `requirements.txt`.
            }
        }
        stage('Run Tests') {
            steps {
                sh '''
                echo "Direktori Sekarang: $(pwd)"
                ls -la
                . venv/bin/activate
                pytest test_example.py
            '''
                //Menampilkan direktori kerja saat ini dan daftar file di dalamnya.
                //Mengaktifkan virtual environment, lalu menjalankan test unit
                //Menggunakan `pytest` pada file `test_example.py`.
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    . venv/bin/activate
                    export PATH=/home/devsecops/sonar-scanner-4.8.0.2856-linux/bin:$PATH
                    sonar-scanner \
                        -Dsonar.projectKey=vulpy \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://192.168.1.11:9000 \
                        -Dsonar.login=${SONAR_TOKEN}
                    '''
                    //Mengaktifkan virtual environment dan menjalankan analisis SonarQube
                    //Menggunakan `sonar-scanner`. Analisis dilakukan untuk proyek `vulpy`
                    //Dengan konfigurasi host dan token autentikasi yang sudah ditentukan.
                }
            }
        }
        stage('Deploy to Staging') {
            when {
                expression { params.BUILD_TYPE == 'Scan + Deploy' }
                //Tahap ini hanya akan dijalankan jika parameter `BUILD_TYPE`
                //Dipilih sebagai "Scan + Deploy".
            }
            steps {
                echo 'Deploying to staging...'
                //Placeholder untuk langkah deployment ke staging.
                //Di sini biasanya mencakup langkah-langkah seperti copy file
                //Atau menjalankan skrip deployment ke server staging.
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished!'
             //Menampilkan pesan penutup setelah pipeline selesai dijalankan.
            //Bagian ini akan dieksekusi meskipun pipeline gagal atau berhasil
        }
    }
}
