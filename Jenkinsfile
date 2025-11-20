pipeline {

    // Ejecuta el pipeline en cualquier nodo disponible
    agent any

    stages {

        stage('Build') {

            // Este stage se ejecuta dentro de un contenedor Docker
            agent {
                docker {
                    // Imagen con Node.js 18 basada en Alpine
                    image 'node:18-alpine'
                    // Mantiene el mismo workspace entre etapas
                    reuseNode true
                }
            }

            steps {
                sh '''
                    # Listar archivos del workspace (solo para depuración)
                    ls -la

                    # Verificar versiones de Node y NPM
                    node --version
                    npm --version

                    # Instalar dependencias usando package-lock.json
                    npm ci

                    # Ejecutar el script de build definido en package.json
                    npm run build

                    # Mostrar archivos generados tras el build
                    ls -la
                '''
            }
        }

        stage('Test') {
            // Este stage también usa un contenedor Node 18
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    # Validar que el build generó el archivo esperado
                    test -f build/index.html

                    # Ejecutar las pruebas del proyecto
                    npm test
                '''
            }
        }
    }

    // =======================
    // SECCIÓN POST-BUILD
    // =======================
    post {

        // Este bloque se ejecuta SIEMPRE, incluso si el pipeline falla
        always {

            // Publica los archivos XML de resultados de pruebas
            // Permite que Jenkins muestre métricas y reportes en la UI
            junit 'test-results/**/*.xml'
        }
    }
}
