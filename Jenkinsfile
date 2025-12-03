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

            # Kill any old process on port 5000
            if lsof -i:5000 -t >/dev/null 2>&1; then
              echo "Killing existing process on port 5000"
              kill -9 $(lsof -i:5000 -t) || true
            fi

            echo "Starting Flask app with python app.py"
            nohup python app.py >/tmp/flask_jenkins.log 2>&1 &

            sleep 3
            echo "Checking if port 5000 is listening..."
            if ! lsof -i:5000 -t >/dev/null 2>&1; then
              echo "Flask did not start, showing log:"
              cat /tmp/flask_jenkins.log || true
              exit 1
            fi
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
