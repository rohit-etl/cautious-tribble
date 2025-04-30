pipeline {
    agent any

    environment {
        NODE_HOME = '/home/selfassessorit/.nvm/versions/node/v22.14.0/bin'
        PATH = "${NODE_HOME}:${env.PATH}"
        EXPECTED_REPO = 'https://github.com/rohit-etl/cautious-tribble.git'
    }

    stages {
        stage('Validate Tag & Repo') {
            steps {
                script {
                    def currentRepo = env.GIT_URL ?: env.GIT_URL_1
                    def tagName = env.GIT_BRANCH ?: ""

                    echo "📦 Repo: ${currentRepo}"
                    echo "🔖 Tag: ${tagName}"

                    if (!(tagName ==~ /^refs\/tags\/dev-.+/)) {
                        error "❌ Not a valid 'dev-*' tag. Aborting."
                    }

                    if (currentRepo != EXPECTED_REPO) {
                        error "❌ Repo mismatch. Expected: ${EXPECTED_REPO}, but got: ${currentRepo}"
                    }

                    def version = tagName.replaceFirst(/^refs\/tags\/dev-/, "")
                    echo "✅ Valid dev tag pushed: Version = ${version}"
                }
            }
        }

        stage('Clone Repository') {
            steps {
                git url: "${env.GIT_URL}", credentialsId: "rohit-etl", refspec: "+refs/tags/*:refs/remotes/origin/tags/*", branch: "${env.GIT_BRANCH.replace('refs/tags/', '')}"
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Build') {
            steps {
                script {
                    try {
                        sh 'npm run build'
                    } catch (err) {
                        echo "Build failed : ${err}"
                        error "❌ Build step failed"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh 'pm2 restart vite-project'
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Deployment failed. Check the logs for more details.'
        }
    }
}