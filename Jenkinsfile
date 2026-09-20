pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('github-pitlens-token')
    }

    stages {
        stage('Build') {
            steps {
                sh './gradlew clean assemble'
            }
        }

        stage('Test') {
            steps {
                sh './gradlew test'
            }
        }

        stage('Calculate Release Version') {
            when {
                branch 'master'
            }

            steps {
                script {
                    sh 'git fetch --tags --force'

                    def latestTag = sh(
                        script: 'git describe --tags --abbrev=0',
                        returnStdout: true
                    ).trim()

                    def currentVersion = latestTag.replaceFirst('^v', '')
                    def parts = currentVersion.tokenize('.')

                    if (parts.size() != 3) {
                        error("Unexpected version format: ${currentVersion}")
                    }

                    def major = parts[0] as int
                    def minor = parts[1] as int
                    def patch = parts[2] as int

                    env.RELEASE_VERSION = "${major}.${minor}.${patch + 1}"

                    echo "Latest release: ${latestTag}"
                    echo "Next release:   ${env.RELEASE_VERSION}"
                }
            }
        }

        stage('Publish') {
            when {
                branch 'master'
            }

            steps {
                withEnv([
                    'GITHUB_ACTOR=someliukh'
                ]) {
                    sh './gradlew -Pversion=$RELEASE_VERSION publish'
                }
            }
        }

        stage('Create Release Tag') {
            when {
                branch 'master'
            }

            steps {
                sh '''
                    git config user.name "PitLens CI"
                    git config user.email "ci@pitlens.local"

                    git tag -a "v$RELEASE_VERSION" \
                        -m "Release $RELEASE_VERSION"

                    git push "https://someliukh:$GITHUB_TOKEN@github.com/PitLens/pitlens-contracts.git" \
                        "v$RELEASE_VERSION"
                '''
            }
        }
    }
}