pipeline {

    agent any

    stages {

        // ======================================
        // BUILD
        // ======================================
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    echo "📁 LISTANDO WORKSPACE"
                    ls -la

                    echo "🔧 VERSIONES"
                    node --version
                    npm --version

                    echo "📦 INSTALANDO DEPENDENCIAS"
                    npm ci

                    echo "🏗️ BUILD DEL PROYECTO"
                    npm run build

                    echo "📁 ARCHIVOS TRAS EL BUILD"
                    ls -la
                '''
            }
        }

        // ======================================
        // TESTS EN PARALELO (Unit + e2e)
        // ======================================
        stage('Tests') {
            parallel {

                // ---- UNIT TESTS ----
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            echo "▶️ Ejecutando Unit Tests"
                            npm test
                        '''
                    }

                    post {
                        always {
                            // JUnit tests XML from Jest
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                // ---- E2E TESTS (Playwright) ----
                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            echo "▶️ Instalando 'serve' para servir build"
                            npm install serve

                            echo "🌐 Sirviendo la aplicación"
                            node_modules/.bin/serve -s build &

                            echo "⏳ Esperando que el server esté arriba"
                            sleep 10

                            echo "🎭 Ejecutando Playwright E2E Tests"
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright HTML Report',
                                allowMissing: false,
                                keepAll: false,
                                alwaysLinkToLastBuild: false,
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
            }
        }

        // ======================================
        // DEPLOY (vacío por ahora—solo CLI demo)
        // ======================================
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "⚙️ Instalando CLI de Netlify (demo)"
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                '''
            }
        }
    }

    // ======================================
    // POST ACTIONS (siempre se ejecutan)
    // ======================================
    post {
        always {
            junit 'test-results/**/*.xml'  // tu parte original
        }
    }
}
