pipeline {
    agent any

    environment {
        APP_NAME = "flash-ci-cd"
        VENV_DIR = ".venv"
    }

    stages {
        stage('Checkout') {
            steps {
                // You already have SCM checkout from Jenkins, so this is optional.
                // If you prefer to rely on that, you can remove this stage entirely.
                checkout scm
            }
        }

        stage('Setup Python Env') {
            steps {
                sh '''
                    set -e
                    python3 --version
                    python3 -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    if [ -f requirements.txt ]; then
                        pip install -r requirements.txt
                    fi
                '''
            }
        }

        stage('Tests') {
            steps {
                sh '''
                    set -e
                    . ${VENV_DIR}/bin/activate
                    pip install pytest >/dev/null 2>&1 || true
                    if [ -d tests ]; then
                        pytest
                    else
                        echo "No tests/ directory found, skipping pytest."
                    fi
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploy step placeholder (e.g., run app, build Docker image, or kubectl apply).'
            }
        }
    }

    post {
        always {
            echo "Pipeline finished with status: ${currentBuild.currentResult}"
        }
    }
}
