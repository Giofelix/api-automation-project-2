pipeline {
    agent any
    
    tools {
        // Asegúrate que este nombre sea igual al que pusiste en Manage Jenkins > Tools
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
                echo "⚙️ Instalando dependencias del proyecto..."
                // Usamos BAT por estar en Windows
                bat 'npm ci'
            }
        }
        
        stage('Ejecución de Pruebas') {
            steps {
                script {
                    echo "🧪 Ejecutando Newman sobre el ambiente: ${params.ENVIRONMENT}"
                    
                    def command = "npx newman run ${COLLECTION} -g ${GLOBAL_ENV} -r cli"
                    
                    if (params.FULL_REPORT) {
                        // Creamos la carpeta de reportes si no existe
                        bat 'if not exist reports mkdir reports'
                        command += ",htmlextra,junit --reporter-htmlextra-export reports/report.html --reporter-junit-export reports/junit.xml"
                    }
                    
                    // Ejecutamos el comando con BAT
                    bat command
                }
            }
        }
        
       stage('Publicación de Resultados') {
            // "post" dentro del stage asegura que esto corra siempre
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