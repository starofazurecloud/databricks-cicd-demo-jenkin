pipeline {
  agent any

  environment {
    DATABRICKS_SHARED_URL  = credentials('adb-shared-url')
    DATABRICKS_PAT         = credentials('databricks-pat')
  }

  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Install Databricks CLI') {
      steps {
        sh '''
          pip install --upgrade pip
          pip install databricks-cli
        '''
      }
    }

    stage('Configure Databricks CLI') {
      steps {
        sh '''
          mkdir -p ~/.databricks

          cat > ~/.databricks/shared.json <<EOF
          {
            "host": "${DATABRICKS_SHARED_URL}",
            "token": "${DATABRICKS_PAT}"
          }
          EOF
        '''
      }
    }

    stage('Deploy Notebooks to Shared Workspace') {
      steps {
        sh '''
          export DATABRICKS_CONFIG_FILE=~/.databricks/shared.json

          for notebook in $(find . -name "*.py"); do
            path=$(echo $notebook | sed 's|^\./||')
            databricks workspace import \
              --language PYTHON \
              --format SOURCE \
              --overwrite \
              "$notebook" \
              "/Shared/$path"
          done
        '''
      }
    }
  }
}
