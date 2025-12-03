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

            pip install pytest

            if [ -d tests ]; then
              # Run pytest but treat exit code 5 (no tests collected) as success
              pytest || ec=$?
              if [ "${ec:-0}" -eq 5 ]; then
                echo "pytest exited with code 5 (no tests collected) – treating as success."
              elif [ "${ec:-0}" -ne 0 ]; then
                echo "pytest failed with exit code ${ec}"
                exit ${ec}
              fi
            else
              echo "No tests/ directory found, skipping pytest."
            fi
        '''
    }
}

       stage('Deploy') {
    steps {
        sh '''
            set -e
            . ${VENV_DIR}/bin/activate
            export FLASK_APP=app.py
            export FLASK_ENV=production
            export FLASK_RUN_HOST=0.0.0.0
            export FLASK_RUN_PORT=5000
            # Kill any previous Flask run on this port
            if lsof -i:5000 -t >/dev/null 2>&1; then
              kill -9 $(lsof -i:5000 -t) || true
            fi
            # Run Flask in background
            nohup flask run >/tmp/flask.log 2>&1 &
            echo "Flask app started on port 5000"
        '''
    }
}

    }

    post {
        always {
            echo "Pipeline finished with status: ${currentBuild.currentResult}"
        }
    }
}
