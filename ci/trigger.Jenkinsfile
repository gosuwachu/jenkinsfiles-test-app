// Option 4A-MB: Trigger Jenkinsfile - orchestrates pipeline jobs (multibranch variant)
// Passes BRANCH_NAME to child jobs so they check out the correct branch/PR

def branchToBuild = env.CHANGE_BRANCH ?: env.BRANCH_NAME

pipeline {
    agent any

    stages {
        stage('Start') {
            steps {
                echo "Starting Mobile CI/CD Pipeline (4A-MB) on branch: ${branchToBuild}"
            }
        }

        stage('Build & Quality') {
            parallel {
                stage('iOS Build') {
                    steps {
                        build job: 'pipeline-4a-mb/ios-build',
                              parameters: [string(name: 'BRANCH_NAME', value: branchToBuild)],
                              wait: true
                    }
                }
                stage('Android Build') {
                    steps {
                        build job: 'pipeline-4a-mb/android-build',
                              parameters: [string(name: 'BRANCH_NAME', value: branchToBuild)],
                              wait: true
                    }
                }
                stage('iOS Tests') {
                    steps {
                        build job: 'pipeline-4a-mb/ios-unit-tests',
                              parameters: [string(name: 'BRANCH_NAME', value: branchToBuild)],
                              wait: true
                    }
                }
                stage('Android Tests') {
                    steps {
                        build job: 'pipeline-4a-mb/android-unit-tests',
                              parameters: [string(name: 'BRANCH_NAME', value: branchToBuild)],
                              wait: true
                    }
                }
                stage('iOS Lint') {
                    steps {
                        build job: 'pipeline-4a-mb/ios-linter',
                              parameters: [string(name: 'BRANCH_NAME', value: branchToBuild)],
                              wait: true
                    }
                }
                stage('Android Lint') {
                    steps {
                        build job: 'pipeline-4a-mb/android-linter',
                              parameters: [string(name: 'BRANCH_NAME', value: branchToBuild)],
                              wait: true
                    }
                }
            }
        }

        stage('Deploy') {
            parallel {
                stage('iOS Deploy') {
                    steps {
                        build job: 'pipeline-4a-mb/ios-deploy',
                              parameters: [string(name: 'BRANCH_NAME', value: branchToBuild)],
                              wait: true
                    }
                }
                stage('Android Deploy') {
                    steps {
                        build job: 'pipeline-4a-mb/android-deploy',
                              parameters: [string(name: 'BRANCH_NAME', value: branchToBuild)],
                              wait: true
                    }
                }
            }
        }
    }
}
