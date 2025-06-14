pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '810cb8d8-caff-4b3b-8205-b272ea1dd08a'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -l
                '''
            }
        }

        stage('Run Tests') {
            parallel{
                    stage('test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo 'Test Stage'
                sh '''
                    test -f build/index.html
                    npm test
                '''
            }

                post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test --reporter=html
                '''
            }

                post {
        always {
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
        }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify-cli --version
                    echo "Deploying to Netlify...Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify-cli status
                    node_modules/.bin/netlify-cli deploy --site-id=${NETLIFY_SITE_ID} --dir=build
                '''
            }
        }

    }

}
