/**
 * Jenkins Declarative Pipeline with:
 * - Static parameters (choice, string, boolean)
 * - Dynamic parameter (BRANCH) using Groovy + git ls-remote
 * - Conditional Test stage (RUN_TESTS)
 * - Environment-based Deploy stage (ENV)
 */

def getGitBranches() {
    // Fetch remote branches from 'origin'
    def output = sh(
        script: "git ls-remote --heads origin | awk '{print \$2}' | sed 's#refs/heads/##'",
        returnStdout: true
    ).trim()

    if (!output) {
        return 'main'
    }

    return output.split('\n').join('\n')
}

pipeline {
    agent any

    // ---- Step 1: Add parameters (choice, string, boolean) ----
    parameters {
        choice(
            name: 'ENV',
            choices: """
dev
qa
prod
""",
            description: 'Target environment for deployment'
        )
        string(
            name: 'APP_VERSION',
            defaultValue: '1.0.0',
            description: 'Application version/tag'
        )
        booleanParam(
            name: 'RUN_TESTS',
            defaultValue: true,
            description: 'Run tests stage?'
        )
        // Placeholder; dynamic values will replace this
        choice(
            name: 'BRANCH',
            choices: "main",
            description: 'Git branch to build (dynamic)'
        )
    }

    stages {

        // ---- Step 2: Use Groovy to fetch dynamic values (branches) ----
        stage('Init dynamic parameters') {
            steps {
                script {
                    echo "Fetching remote branches for dynamic parameter..."
                    def branchChoices = getGitBranches()
                    echo "Available branches:\n${branchChoices}"

                    // Re-define pipeline parameters dynamically
                    properties([
                        parameters([
                            choice(
                                name: 'ENV',
                                choices: "dev\nqa\nprod",
                                description: 'Target environment for deployment'
                            ),
                            string(
                                name: 'APP_VERSION',
                                defaultValue: APP_VERSION,
                                description: 'Application version/tag'
                            ),
                            booleanParam(
                                name: 'RUN_TESTS',
                                defaultValue: RUN_TESTS,
                                description: 'Run tests stage?'
                            ),
                            choice(
                                name: 'BRANCH',
                                choices: branchChoices,
                                description: 'Git branch to build (dynamic)'
                            )
                        ])
                    ])
                }
            }
        }

        stage('Checkout') {
            steps {
                echo "Checking out branch: ${BRANCH}"
                checkout scm
                sh "git checkout ${BRANCH}"
            }
        }

        stage('Build') {
            steps {
                echo "Building app version ${APP_VERSION} for ENV=${ENV}"
                sh "./app.sh"
            }
        }

        // ---- Step 3: Conditional stage based on parameter value ----
        stage('Test') {
            when {
                expression { return RUN_TESTS == true }
            }
            steps {
                echo "RUN_TESTS is true, executing tests..."
                sh "./test.sh"
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying app version ${APP_VERSION} to ${ENV} from branch ${BRANCH}"
                sh """
                  echo 'Simulating deployment...'
                  echo 'ENV=${ENV}, VERSION=${APP_VERSION}, BRANCH=${BRANCH}'
                """
            }
        }
    }

    post {
        always {
            echo "Build finished. Summary:"
            echo "  ENV        = ${ENV}"
            echo "  APP_VERSION= ${APP_VERSION}"
            echo "  BRANCH     = ${BRANCH}"
            echo "  RUN_TESTS  = ${RUN_TESTS}"
        }
    }
}

