def getGitBranches() {
    def output = sh(
        script: "git ls-remote --heads origin | awk '{print \$2}' | sed 's#refs/heads/##'",
        returnStdout: true
    ).trim()

    return output ? output.split('\n').join('\n') : 'main'
}

pipeline {
    agent any

    parameters {
        choice(
            name: 'ENV',
            choices: """
dev
qa
prod
""",
            description: 'Target environment'
        )
        string(
            name: 'APP_VERSION',
            defaultValue: '1.0.0',
            description: 'App version'
        )
        booleanParam(
            name: 'RUN_TESTS',
            defaultValue: true,
            description: 'Run tests?'
        )
        choice(
            name: 'BRANCH',
            choices: "main",
            description: 'Git branch (dynamic)'
        )
    }

    stages {

        stage("Init dynamic parameters") {
            steps {
                script {
                    echo "Fetching dynamic git branches..."
                    def branchChoices = getGitBranches()

                    properties([
                        parameters([
                            choice(
                                name: 'ENV',
                                choices: "dev\nqa\nprod",
                                description: ''
                            ),
                            string(
                                name: 'APP_VERSION',
                                defaultValue: params.APP_VERSION,
                                description: ''
                            ),
                            booleanParam(
                                name: 'RUN_TESTS',
                                defaultValue: params.RUN_TESTS,
                                description: ''
                            ),
                            choice(
                                name: 'BRANCH',
                                choices: branchChoices,
                                description: ''
                            )
                        ])
                    ])
                }
            }
        }

        stage('Checkout') {
            steps {
                echo "Checking out branch: ${params.BRANCH}"
                checkout scm
                sh "git checkout ${params.BRANCH}"
            }
        }

        stage('Build') {
            steps {
                echo "Building app ${params.APP_VERSION} for ENV=${params.ENV}"
                sh "./app.sh"
            }
        }

        stage('Test') {
            when {
                expression { params.RUN_TESTS == true }
            }
            steps {
                echo "Running tests..."
                sh "./test.sh"
            }
        }

        stage('Deploy') {
            steps {
                sh """
                  echo 'Deploying...'
                  echo 'ENV=${params.ENV}'
                  echo 'VERSION=${params.APP_VERSION}'
                  echo 'BRANCH=${params.BRANCH}'
                """
            }
        }
    }

    post {
        always {
            echo "Build Summary:"
            echo "ENV        = ${params.ENV}"
            echo "APP_VERSION= ${params.APP_VERSION}"
            echo "BRANCH     = ${params.BRANCH}"
            echo "RUN_TESTS  = ${params.RUN_TESTS}"
        }
    }
}

