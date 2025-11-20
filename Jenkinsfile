pipeline {

    agent any

    environment {
        NETLIFY_SITE_ID = "cd5fae46-e99a-46af-862e-b9ca471784b6"
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm ci
                    npm run build
                '''
            }
        }

        stage('Tests') {
            parallel {

                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh 'npm test'
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install --no-save serve

                            node_modules/.bin/serve -s build &
                            sleep 10

                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
                        }
                    }
                }
            }
        }

        stage('Deploy to Netlify') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install -g netlify-cli
                    netlify --version

                    echo "🚀 Deploying to Netlify..."

                    netlify deploy \
                      --prod \
                      --site $NETLIFY_SITE_ID \
                      --auth $NETLIFY_AUTH_TOKEN \
                      --dir=build
                '''
            }
        }

    }

    post {
        always {
            junit 'test-results/**/*.xml'
        }
    }
}
