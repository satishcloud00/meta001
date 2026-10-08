pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate JSON') {
            steps {
                sh '''
                    echo "Validating JSON files..."

                    find environment node -name "*.json" -print0 | while IFS= read -r -d '' file
                    do
                        python3 -m json.tool "$file" > /dev/null
                        echo "Valid JSON: $file"
                    done
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'

                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=meta001 \
                            -Dsonar.projectName=meta001 \
                            -Dsonar.sources=.
                        """
                    }
                }
            }
        }
    }
}
