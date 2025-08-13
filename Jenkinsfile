// Inicia la definición del pipeline
pipeline {
    // Define en qué agente (máquina) de Jenkins se ejecutará. 'any' es el más simple.
    agent any

    // Define los parámetros que este pipeline puede recibir.
    parameters {
        string(name: 'IMAGE_NAME')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Tag de la imagen a desplegar')
    }

    // Define las etapas (etapas) del pipeline.
    stages {
        // --- ETAPA 1: Desplegar en Kubernetes ---
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Imprime un mensaje en la consola de Jenkins para saber qué se está desplegando.
                    // Usa los parámetros recibidos (${params.IMAGE_NAME}, ${params.IMAGE_TAG}).
                    echo "Iniciando despliegue para la imagen: ${params.IMAGE_NAME}:${params.IMAGE_TAG}"

                    // 1. ACTUALIZAR EL MANIFIESTO DE DEPLOYMENT
                    // 'sh' ejecuta un comando de shell.
                    // Usamos el comando 'sed' para buscar y reemplazar la línea de la imagen en el archivo deployment.yaml.
                    // La sintaxis sed -i 's|patrón_a_buscar|texto_de_reemplazo|' deployment.yaml funciona así:
                    //   -i: Edita el archivo "in-place" (directamente).
                    //   s|...|...|: Comando de sustitución. Usamos '|' como separador porque la URL contiene '/'.
                    //   image: .*: Busca una línea que empiece con "image: " seguida de cualquier carácter (.*).
                    echo "Actualizando el archivo deployment.yaml..."
                    sh "sed -i 's|image: .*|image: ${params.IMAGE_NAME}:${params.IMAGE_TAG}|' deployment.yaml"
                    
                    echo "Contenido de deployment.yaml actualizado:"
                    sh "cat deployment.yaml"

                    // 2. APLICAR LOS CAMBIOS AL CLÚSTER
                    // Ejecuta kubectl apply para todos los archivos .yaml en el directorio actual.
                    echo "Aplicando manifiestos a k3s..."
                    sh "sudo kubectl apply -f ."

                    // 3. FORZAR LA ACTUALIZACIÓN DE LOS PODS
                    echo "Forzando el reinicio del deployment 'frontend-deployment'..."
                    sh "sudo kubectl rollout restart deployment frontend-deployment -n production"

                    echo "¡Despliegue completado exitosamente!"
                }
            }
        }
    }

    // 'post' define acciones que se ejecutan después de que el pipeline termine.
    post {
        // 'always' se ejecuta siempre, sin importar si el pipeline falló o tuvo éxito.
        always {
            // Limpia el workspace para la siguiente ejecución. Es una buena práctica.
            echo "Limpiando el workspace de Jenkins..."
            cleanWs()
        }
    }
}
