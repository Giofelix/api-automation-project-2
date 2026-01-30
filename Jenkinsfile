pipeline {
    agent any

    // PUNTO 2 y 4: Disparadores automáticos
    triggers {
        pollSCM('H/5 * * * *') // Revisa GitHub cada 5 min. Si hay cambios, ejecuta.
        cron('H 08 * * *')     // Se ejecuta todos los días a las 8:00 AM automáticamente.
    }

    tools {
        nodejs 'node20' // Debe coincidir con tu nombre en 'Global Tool Configuration'
    }

    environment {
        COLLECTION = 'collections/API_Automation_Project_collection.json'
        GLOBAL_ENV = 'environments/workspace.postman_globals.json'
    }

    stages {
        stage('🚀 Inicializando') {
            steps {
                echo "--- Iniciando Proceso de Automatización ---"
                deleteDir() // Limpia la carpeta para que no haya archivos viejos
                checkout scm
            }
        }

        stage('📦 Instalando Herramientas') {
            steps {
                echo "Instalando dependencias necesarias..."
                bat 'npm ci'
            }
        }

        stage('🧪 Ejecutando Pruebas API') {
            steps {
                script {
                    echo "Ejecutando Newman... Por favor espera."
                    // Creamos la carpeta de reportes
                    bat 'if not exist reports mkdir reports'
                    
                    // Ejecución silenciosa para que la consola no sea un desorden
                    // Usamos --suppress-exit-code para que el reporte se genere SIEMPRE
                    bat "npx newman run ${COLLECTION} -g ${GLOBAL_ENV} -r cli,htmlextra --reporter-htmlextra-export reports/reporte_final.html --suppress-exit-code"
                }
            }
        }

        stage('📊 Generando Reporte') {
            steps {
                echo "Publicando resultados en la interfaz de Jenkins..."
                // PUNTO 1: Configuración del reporte HTML
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'reporte_final.html',
                    reportName: 'Reporte Newman HTML'
                ])
            }
        }
    }

    post {
        always {
            echo "--- Finalizado: Puedes ver el reporte en el menú de la izquierda ---"
        }
    }
}