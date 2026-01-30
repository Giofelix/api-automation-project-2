pipeline {
    agent any
    
    tools {
        // Debe coincidir con el nombre que configuraste en "Global Tool Configuration"
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
                // Jenkins descargará tu repo de GitHub automáticamente
                checkout scm
            }
        }
        
        stage('Instalación') {
            steps {
                echo "⚙️ Instalando dependencias del proyecto..."
                sh 'npm ci'
            }
        }
        
        stage('Ejecución de Pruebas') {
            steps {
                script {
                    echo "🧪 Ejecutando Newman sobre el ambiente: ${params.ENVIRONMENT}"
                    
                    // Nota el uso de -g para tus archivos globales
                    def command = "npx newman run ${COLLECTION} -g ${GLOBAL_ENV} -r cli"
                    
                    if (params.FULL_REPORT) {
                        command += ",htmlextra,junit --reporter-htmlextra-export reports/report.html --reporter-junit-export reports/junit.xml"
                    }
                    
                    sh command
                }
            }
        }
        
        stage('Publicación de Resultados') {
            steps {
                // Muestra el reporte HTML dentro de la interfaz de Jenkins
                publishHTML([
                    reportDir: 'reports',
                    reportFiles: 'report.html',
                    reportName: 'Reporte HTML de Newman',
                    keepAll: true
                ])
                
                // Muestra gráficas de fallos/éxitos
                junit 'reports/junit.xml'
            }
        }
    }
    
    post {
        always {
            echo "🏁 Proceso finalizado. Limpiando espacio de trabajo..."
            cleanWs()
        }
    }
}