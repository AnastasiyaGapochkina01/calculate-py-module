pipeline {
  agent { label 'ecom' }

  parameters {
    booleanParam(name: 'RUN_TESTS', defaultValue: true, description: '')
  }

  environment {
    BASE_DIR = "app"
    PRJ_NAME = "calc"
    GIT_URL = "git@github.com:AnastasiyaGapochkina01/calculate-py-module.git"
  }

  stages {
    stage('Prepare host') {
      steps {
        script {
          sh """
            sudo apt install python3.12-venv -y > /dev/null
          """
        }
      }
    }
    stage('Clone repo') {
      steps {
        script {
          sh """
            if [ -d ${env.BASE_DIR}/${env.PRJ_NAME}/.git ]; then
              echo "Repo exists. Updating..."
              cd ${env.BASE_DIR}/${env.PRJ_NAME}
              git pull
            else
              echo "Repo absent. Clonning..."
              mkdir -p ${env.BASE_DIR}
              git clone ${env.GIT_URL} ${env.BASE_DIR}/${env.PRJ_NAME}
            fi
          """
        }
      }
    }
    stage('Setup Environment') {
      steps {
        script {
          sh """
            cd ${env.BASE_DIR}/${env.PRJ_NAME}
            python3 -m venv venv
            . venv/bin/activate
            pip install --upgrade pip
            pip install -e .
            pip install -r requirements.txt
          """
        }
      }
    }
    stage('Security Scan') {
      steps {
        script {
          sh """
            cd ${env.BASE_DIR}/${env.PRJ_NAME}
            . venv/bin/activate
            bandit -r app/ -f json -o bandit_results.json || true
          """
        }
      }
    }
    stage('Run Unit Tests') {
      when {
        expression { params.RUN_TESTS }
      }
      steps {
        script {
          sh """
            cd ${env.BASE_DIR}/${env.PRJ_NAME}
            . venv/bin/activate
            PYTHONPATH=src pytest -v tests/ --junitxml=test-results.xml
          """
        }
      }
    }
    stage('Package Application') {
      steps {
        script {
          sh """
            cd ${env.BASE_DIR}/${env.PRJ_NAME}
            . venv/bin/activate
            python setup.py sdist bdist_wheel
            ls -la dist/ || echo "Папка dist не существует"
          """
        }
      }
    }
  }
}
