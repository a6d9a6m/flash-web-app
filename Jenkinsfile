pipeline {
    agent any

    environment {
        // 假设 Jenkins 容器里已经装了 python3
        PYTHON_PATH = 'python3'
    }

    stages {
        stage('Verify Python Path') {
            steps {
                sh """
                    echo "=== 验证 Python 路径及版本 ==="
                    ${PYTHON_PATH} --version
                    echo "Python 路径验证通过！"
                """
            }
        }

        stage('Checkout') {
            steps {
                git url: 'https://github.com/zhangping99/myflaskapp.git', branch: 'main'
            }
        }

        stage('Fix pip') {
            steps {
                sh """
                    echo "=== 修复 pip 环境 ==="
                    ${PYTHON_PATH} -m ensurepip --upgrade || true
                    ${PYTHON_PATH} -m pip install --upgrade pip
                    ${PYTHON_PATH} -m pip --version
                """
            }
        }

        stage('Install Dependencies') {
            steps {
                sh """
                    echo "=== 安装依赖 ==="
                    ${PYTHON_PATH} -m pip install -r requirements.txt
                """
            }
        }

        stage('Lint') {
            steps {
                sh """
                    ${PYTHON_PATH} -m pip install flake8
                    ${PYTHON_PATH} -m flake8 app.py tests/
                """
            }
        }

        stage('Test') {
            steps {
                sh """
                    ${PYTHON_PATH} -m pip install pytest pytest-cov
                    ${PYTHON_PATH} -m pytest --cov=app tests/ --cov-report=html
                """
            }
            post {
                always {
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'htmlcov',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }

        stage('Build') {
            steps {
                sh """
                    ${PYTHON_PATH} -m pip install pyinstaller
                    ${PYTHON_PATH} -m pyinstaller --onefile app.py
                """
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh """
                    nohup ${PYTHON_PATH} app.py > app.log 2>&1 &
                """
            }
        }
    }

    post {
        success { echo 'CI/CD pipeline completed successfully!' }
        failure { echo 'CI/CD pipeline failed!' }
    }
}
