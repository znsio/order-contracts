pipeline {
    agent any

    tools {
        nodejs 'NodeJS'  // Make sure you have NodeJS configured in Jenkins Global Tool Configuration
    }

    environment {
        // Define environment variables
        SPECMATIC_ORG_ID = "66fe6c555e232d36a28fef94"  // Stored in Jenkins credentials
        WORKSPACE = pwd()  // Get Jenkins workspace path
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('List Files') {
            steps {
                script {
                    sh 'ls -R'
                }
            }
        }

        stage('Setup') {
            steps {
                script {
                    // Install the reporter package globally
                    sh 'npm install -g specmatic-insights-github-build-reporter'
                }
            }
        }
        
        stage('Run OpenAPI Backward compatibility Check') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') { // have to handle this because bcc is failing build when no files are changed
                        sh '''
                            java -jar specmatic_2.0.33.jar backward-compatibility-check --base-branch origin/main
                        '''
                    }
                }
            }
        }

        stage('RGenerate central contract repo report') {
            steps {
                script {
                    sh '''
                        java -jar specmatic_2.0.33.jar central-contract-repo-report
                    '''
                }
            }
        }

        stage('Debug Values') {
            steps {
                script {
                    echo """
                    Debugging all values:
                    ---------------------
                    WORKSPACE: ${WORKSPACE}
                    SPECMATIC_ORG_ID: ${SPECMATIC_ORG_ID}
                    JOB_NAME: ${env.JOB_NAME}
                    BUILD_NUMBER: ${env.BUILD_NUMBER}
                    BRANCH_NAME: ${env.BRANCH_NAME ?: 'main (fallback)'}
                    
                    GIT Info:
                    ---------------------
                    """
                    
                    // Get Git URL safely
                    def gitUrl = sh(script: 'git config --get remote.origin.url || echo "No Git URL found"', returnStdout: true).trim()
                    echo "GIT URL: ${gitUrl}"
                    
                    // Check reports directory
                    sh """
                        echo "Reports Directory Check:"
                        echo "------------------------"
                        ls -la ${WORKSPACE}/build/reports/specmatic || echo "Reports directory not found"
                    """
                }
            }
        }

        stage('Upload Specmatic Insights Reports') {
            steps {
                script {
                    sh """
                        specmatic-insights-github-build-reporter \
                        --specmatic-insights-host https://insights.specmatic.in \
                        --specmatic-reports-dir pwd()/build/reports/specmatic \
                        --org-id YOUR_SPECMATIC_ORG_ID \
                        --branch-ref refs/heads/${env.BRANCH_NAME ?: 'main'} \
                        --branch-name ${env.BRANCH_NAME ?: 'main'} \
                        --build-definition-id "${env.JOB_NAME}" \
                        --build-id ${env.BUILD_NUMBER} \
                        --repo-name "${env.GIT_REPO_NAME}" \
                        --repo-id "${env.GIT_REPO_ID}" \
                        --repo-url \$(git config --get remote.origin.url || echo 'undefined')
                    """
                }
            }
            // steps {
            //     script {
            //         def githubToken = env.GH_REPOSITORY_TOKEN
            //         def orgId = '66fe6c555e232d36a28fef94'
            //         def branchRef = env.GIT_BRANCH
            //         def branchName = env.GIT_BRANCH
            //         def buildId = env.BUILD_ID
            //         def repoName = env.GIT_REPO_NAME
            //         def repoId = env.GIT_REPO_ID
            //         def repoUrl = env.GIT_URL

            //         def jsonPayload = """
            //         {
            //             "github-token": "${githubToken}",
            //             "org-id": "${orgId}",
            //             "branch-ref": "${branchRef}",
            //             "branch-name": "${branchName}",
            //             "build-id": "${buildId}",
            //             "repo-name": "${repoName}",
            //             "repo-id": "${repoId}",
            //             "repo-url": "${repoUrl}"
            //         }
            //         """

            //         sh """
            //         curl -X POST -H "Content-Type: application/json" \
            //         -d '${jsonPayload}' \
            //         https://insights.specmatic.io/report
            //         """
            //     }
            // }
        }
    }
}
