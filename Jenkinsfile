pipeline {
    agent any

    triggers {
        pollSCM('H/3 * * * *')   
    }

    options {
        timeout(time: 15, unit: 'MINUTES')           
        buildDiscarder(logRotator(numToKeepStr: '10')) 
    }

    environment {
        PORT = '4000'
        COMPOSE_PROJECT_NAME = "aeco22food"       
    }

    stages {
        stage('Cleanup trước khi build') {
            steps {
                script {
                    
                    sh '''
                    docker-compose down --remove-orphans --volumes --rmi local || true
                    docker system prune -f || true
                    docker volume prune -f || true
                    '''
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/vathcodes/aeco22food.git']],
                    extensions: [ [$class: 'CleanBeforeCheckout'] ]  
                )
            }
        }

        stage('Build & Deploy') {
            steps {
                withCredentials([
                    string(credentialsId: 'MONGODB_URI', variable: 'MONGODB_URI'),
                    string(credentialsId: 'JWT_SECRET', variable: 'JWT_SECRET'),
                    string(credentialsId: 'VNPAY_TMN_CODE', variable: 'VNPAY_TMN_CODE'),
                    string(credentialsId: 'VNPAY_HASH_SECRET', variable: 'VNPAY_HASH_SECRET')
                ]) {
                    sh '''
                    # Tạo .env
                    cat > .env <<EOF
                PORT=$PORT
                MONGODB_URI=$MONGODB_URI
                JWT_SECRET=$JWT_SECRET
                VNPAY_TMN_CODE=$VNPAY_TMN_CODE
                VNPAY_HASH_SECRET=$VNPAY_HASH_SECRET
                BACKEND_URL=http://34.9.54.54:4000
                FRONTEND_URL=http://34.9.54.54:3000
                EOF

                    # Build nhanh nhất có thể + dùng cache BuildKit
                    docker-compose build --parallel --pull
                    docker-compose up -d --force-recreate

                    # Dọn dẹp lại lần cuối cho sạch sẽ
                    docker image prune -f || true
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker-compose down --remove-orphans || true'
        }
        success {
            echo 'Deploy thành công! Ứng dụng đang chạy ngon lành tại http://34.9.54.54'
        }
        failure {
            echo 'Build thất bại – đã tự động dọn dẹp sạch sẽ'
        }
    }
}