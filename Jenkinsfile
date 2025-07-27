pipeline {
    agent any

    environment {
        ADB_DEV_URL     = credentials('adb-dev-url')
        ADB_SHARED_URL  = credentials('adb-shared-url')
        DATABRICKS_TOKEN = credentials('databricks-pat')  // stored in Azure Key Vault
    }

    stages {
      stage('Generate databricks.cfg') {
            steps {
                script {
                    def databricksCfgContent = """
                    [DEFAULT]
                    host = ${env.DATABRICKS_HOST}
                    token = ${env.DATABRICKS_TOKEN}
                    """.stripIndent()

                    writeFile file: 'databricks.cfg', text: databricksCfgContent
                    echo '✅ databricks.cfg file generated successfully.'
                }
            }
        }

        stage('Verify Config') {
            steps {
                sh 'cat databricks.cfg'
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Export Notebooks from Dev Workspace') {
            steps {
                sh '''
                databricks workspace export_dir \
                  --profile DEV \
                  --workspace-dir /Workspace/Users/starofcloud@outlook.com/databricks-cicd-demo-jenkins \
                  --output-dir notebooks
                '''
            }
        }

        stage('Import Notebooks to Shared Workspace') {
            steps {
                sh '''
                databricks workspace import_dir \
                  --profile SHARED \
                  --source-path notebooks \
                  --workspace-path /Workspace/Shared/databricks-cicd-demo-jenkins \
                  --overwrite
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
