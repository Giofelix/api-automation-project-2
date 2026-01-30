pipeline {
    agent any
    
    tools {
        nodejs 'node20' 
    }
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['DEV', 'STAGING', 'PROD'], description: 'Selecciona el ambiente')
        booleanParam(name: 'FULL_REPORT', defaultValue: true, description: '¿Generar reporte HTML?')
    }
    
    environment {
        COLLECTION = 'collections/API_Automation_Project_collection.json'
        GLOBAL_ENV = 'environments/workspace.postman_globals.json'
    }
    
    stages {
        stage('Descarga de Código') {
            steps {
                checkout scm
            }
        }
        
        stage('Instalación') {
            steps {
                echo "⚙️ Instalando dependencias..."
                bat 'npm ci'
            }
        }
        
        stage('Ejecución de Pruebas') {
            steps {
                script {
                    echo "🧪 Ejecutando Newman sobre el ambiente: ${params.ENVIRONMENT}"
                    
                    def command = "npx newman run ${COLLECTION} -g ${GLOBAL_ENV} -r cli"
                    
                    if (params.FULL_REPORT) {
                        bat 'if not exist reports mkdir reports'
                        command += ",htmlextra,junit --reporter-htmlextra-export reports/report.html --reporter-junit-export reports/junit.xml"
                    }
                    
                    bat command
                }
            }
        }
        
        stage('Publicación de Resultados') {
            steps {
                echo "📊 Procesando reportes de prueba..."
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'reports',
                        reportFiles: 'report.html',
                        reportName: 'Reporte HTML de Newman'
                    ])
                    
                    junit 'reports/junit.xml'
                }
            }
        }
    }
    
    post {
        always {
            echo "🏁 Proceso finalizado."
            cleanWs()
        }
    }
}