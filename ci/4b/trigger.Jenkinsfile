// Option 4B-MB: Trigger Jenkinsfile - orchestrates pipeline jobs (multibranch variant)
// Pure orchestrator — child jobs publish their own commit statuses
// No publishChecks here; each child job handles its own GitHub status reporting
// PR diff filtering: only triggers iOS or Android jobs based on changed files

import groovy.transform.Field

@Field GITHUB_OWNER = 'gosuwachu'
@Field GITHUB_REPO = 'jenkinsfiles-test-app'

@Field IOS_CONTEXTS = [
    'ci/ios-build (4B-MB)',
    'ci/ios-unit-tests (4B-MB)',
    'ci/ios-linter (4B-MB)',
    'ci/ios-deploy (4B-MB)',
]

@Field ANDROID_CONTEXTS = [
    'ci/android-build (4B-MB)',
    'ci/android-unit-tests (4B-MB)',
    'ci/android-linter (4B-MB)',
    'ci/android-deploy (4B-MB)',
]

def detectPlatforms() {
    if (!env.CHANGE_TARGET) {
        echo 'Not a PR build — running all platforms'
        return [ios: true, android: true]
    }

    try {
        sh "git fetch origin ${env.CHANGE_TARGET} --quiet"
        def changedFiles = sh(
            script: "git diff --name-only origin/${env.CHANGE_TARGET}...HEAD",
            returnStdout: true
        ).trim()

        if (!changedFiles) {
            echo 'No changed files detected — running all platforms'
            return [ios: true, android: true]
        }

        echo "Changed files in PR:\n${changedFiles}"

        def hasIos = false
        def hasAndroid = false
        def hasOther = false

        changedFiles.split('\n').each { file ->
            if (file.startsWith('ios/')) {
                hasIos = true
            } else if (file.startsWith('android/')) {
                hasAndroid = true
            } else {
                hasOther = true
            }
        }

        if (hasOther) {
            echo 'Files outside ios/ and android/ changed — running all platforms'
            return [ios: true, android: true]
        }

        echo "Platform detection: iOS=${hasIos}, Android=${hasAndroid}"
        return [ios: hasIos, android: hasAndroid]
    } catch (e) {
        echo "WARNING: Failed to detect changed files: ${e.message}. Running all platforms."
        return [ios: true, android: true]
    }
}

def publishSkippedStatuses(List<String> contexts, String platform) {
    def sha = env.GIT_COMMIT ?: sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
    withCredentials([usernamePassword(credentialsId: 'github-app',
            usernameVariable: 'GH_APP', passwordVariable: 'GH_TOKEN')]) {
        contexts.each { context ->
            echo "Publishing skipped status for: ${context}"
            sh """curl -s -X POST \
                -H "Authorization: token \$GH_TOKEN" \
                -H "Accept: application/vnd.github+json" \
                "https://api.github.com/repos/${GITHUB_OWNER}/${GITHUB_REPO}/statuses/${sha}" \
                -d '{"state":"success","context":"${context}","description":"Skipped — no ${platform} changes","target_url":"${env.BUILD_URL}"}'"""
        }
    }
}

pipeline {
    agent any

    environment {
        BRANCH_TO_BUILD = "${env.CHANGE_BRANCH ?: env.BRANCH_NAME}"
    }

    stages {
        stage('Start') {
            steps {
                script {
                    echo "Starting Mobile CI/CD Pipeline (4B-MB) on branch: ${env.BRANCH_TO_BUILD}"

                    def platforms = detectPlatforms()
                    env.RUN_IOS = platforms.ios.toString()
                    env.RUN_ANDROID = platforms.android.toString()

                    echo "Will run — iOS: ${env.RUN_IOS}, Android: ${env.RUN_ANDROID}"

                    if (env.RUN_IOS != 'true') {
                        publishSkippedStatuses(IOS_CONTEXTS, 'iOS')
                    }
                    if (env.RUN_ANDROID != 'true') {
                        publishSkippedStatuses(ANDROID_CONTEXTS, 'Android')
                    }
                }
            }
        }

        stage('Build & Quality') {
            parallel {
                stage('iOS Build') {
                    when { expression { env.RUN_IOS == 'true' } }
                    steps {
                        build job: 'pipeline-4b-mb/ios-build',
                              parameters: [string(name: 'BRANCH_NAME', value: env.BRANCH_TO_BUILD)],
                              wait: true
                    }
                }
                stage('Android Build') {
                    when { expression { env.RUN_ANDROID == 'true' } }
                    steps {
                        build job: 'pipeline-4b-mb/android-build',
                              parameters: [string(name: 'BRANCH_NAME', value: env.BRANCH_TO_BUILD)],
                              wait: true
                    }
                }
                stage('iOS Tests') {
                    when { expression { env.RUN_IOS == 'true' } }
                    steps {
                        build job: 'pipeline-4b-mb/ios-unit-tests',
                              parameters: [string(name: 'BRANCH_NAME', value: env.BRANCH_TO_BUILD)],
                              wait: true
                    }
                }
                stage('Android Tests') {
                    when { expression { env.RUN_ANDROID == 'true' } }
                    steps {
                        build job: 'pipeline-4b-mb/android-unit-tests',
                              parameters: [string(name: 'BRANCH_NAME', value: env.BRANCH_TO_BUILD)],
                              wait: true
                    }
                }
                stage('iOS Lint') {
                    when { expression { env.RUN_IOS == 'true' } }
                    steps {
                        build job: 'pipeline-4b-mb/ios-linter',
                              parameters: [string(name: 'BRANCH_NAME', value: env.BRANCH_TO_BUILD)],
                              wait: true
                    }
                }
                stage('Android Lint') {
                    when { expression { env.RUN_ANDROID == 'true' } }
                    steps {
                        build job: 'pipeline-4b-mb/android-linter',
                              parameters: [string(name: 'BRANCH_NAME', value: env.BRANCH_TO_BUILD)],
                              wait: true
                    }
                }
            }
        }

        stage('Deploy') {
            parallel {
                stage('iOS Deploy') {
                    when { expression { env.RUN_IOS == 'true' } }
                    steps {
                        build job: 'pipeline-4b-mb/ios-deploy',
                              parameters: [string(name: 'BRANCH_NAME', value: env.BRANCH_TO_BUILD)],
                              wait: true
                    }
                }
                stage('Android Deploy') {
                    when { expression { env.RUN_ANDROID == 'true' } }
                    steps {
                        build job: 'pipeline-4b-mb/android-deploy',
                              parameters: [string(name: 'BRANCH_NAME', value: env.BRANCH_TO_BUILD)],
                              wait: true
                    }
                }
            }
        }
    }
}
