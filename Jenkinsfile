
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
                    set -e
                    find environment node -name "*.json" -print0 |
                    while IFS= read -r -d '' file
                    do
                        python3 -m json.tool "$file" > /dev/null
                        echo "Valid JSON: $file"
                    done
                '''
            }
        }

        stage('Create or Reuse Pull Request') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-credentials',
                    usernameVariable: 'GITHUB_USER',
                    passwordVariable: 'GITHUB_TOKEN'
                )]) {
                    script {
                        def prNumber = sh(
                            returnStdout: true,
                            script: '''
                                set -e
                                export GH_TOKEN="$GITHUB_TOKEN"

                                PR_NUMBER=$(gh pr list \
                                  --repo satishcloud00/meta001 \
                                  --head feature \
                                  --base main \
                                  --state open \
                                  --json number \
                                  --jq '.[0].number // empty')

                                if [ -z "$PR_NUMBER" ]; then
                                    gh pr create \
                                      --repo satishcloud00/meta001 \
                                      --base main \
                                      --head feature \
                                      --title "Changes from feature branch" \
                                      --body "Automated PR created by Jenkins" \
                                      > /dev/null

                                    PR_NUMBER=$(gh pr list \
                                      --repo satishcloud00/meta001 \
                                      --head feature \
                                      --base main \
                                      --state open \
                                      --json number \
                                      --jq '.[0].number // empty')
                                fi

                                test -n "$PR_NUMBER"
                                echo "$PR_NUMBER"
                            '''
                        ).trim()

                        echo "Processing PR #${prNumber}"

                        build job: 'pr-validation',
                            parameters: [
                                string(name: 'PR_NUMBER', value: prNumber)
                            ],
                            wait: true
                    }
                }
            }
        }
    }
}
