pipeline {

    agent any

    environment {
        NETLIFY_SITE_ID  = "cd5fae46-e99a-46af-862e-b9ca471784b6"
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        CI = "true"
    }

    stages {

        /* ============================
         * BUILD
         * ============================ */
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
                stash includes: 'build/**', name: 'build'
                stash includes: 'node_modules/**', name: 'node_modules_alpine'
            }
        }

        /* ============================
         * TESTS (UNIT + E2E)
         * ============================ */
        stage('Tests') {
            parallel {

                /* --- UNIT TESTS --- */
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        unstash 'node_modules_alpine'
                        sh 'npm test'
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                /* --- E2E TESTS (PLAYWRIGHT) --- */
                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        unstash 'build'

                        sh '''
                            # El node_modules alpino NO sirve aquí → debes reinstalar
                            npm ci

                            npx serve -s build & 
                            sleep 8

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

        /* ============================
         * DEPLOY
         * ============================ */
        stage('Deploy to Netlify') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                unstash 'build'

                sh '''
                    npm install netlify-cli --no-save
                    echo "🚀 Deploying to Netlify…"

                    npx netlify deploy \
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
