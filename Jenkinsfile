pipeline {
    agent { label 'dev-server' }

    environment {
        NODE_HOME = '/home/selfassessorit/.nvm/versions/node/v22.14.0/bin'
        PATH = "${NODE_HOME}:${env.PATH}"
        EXPECTED_REPO = 'https://github.com/rohit-etl/cautious-tribble.git'
    }

    options {
        disableConcurrentBuilds()
    }

    stages {

        stage('Validate Tag & Repo') {
            steps {
                script {
                    def currentRepo = env.GIT_URL ?: env.GIT_URL_1
                    def tagRef = env.GIT_BRANCH ?: ""

                    echo "📦 Repo: ${currentRepo}"
                    echo "🔖 Branch/Tag: ${tagRef}"

                    if (!(tagRef ==~ /^refs\/tags\/dev-.+/)) {
                        error "❌ Not a valid 'dev-*' tag. Aborting."
                    }

                    if (currentRepo != EXPECTED_REPO) {
                        error "❌ Repo mismatch. Expected: ${EXPECTED_REPO}, but got: ${currentRepo}"
                    }

                    // Extract version
                    def version = tagRef.replaceFirst(/^refs\/tags\/dev-/, "")
                    env.VERSION = version
                    env.TAG_REF = tagRef.replaceFirst(/^refs\/tags\//, "")
                    echo "✅ Valid dev tag pushed: Version = ${version}"
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "refs/tags/${env.TAG_REF}"]],
                        userRemoteConfigs: [[
                            url: "${env.GIT_URL}",
                            credentialsId: "rohit-etl",
                            refspec: "+refs/tags/*:refs/remotes/origin/tags/*"
                        ]]
                    ])
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build location') {
            steps {
                sh 'pwd'
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

        stage('Start Production') {
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
