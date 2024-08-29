
# Implementa pruebas de rendimiento al ciclo de integración continua utilizando JMeter.

## Índice
1. [Introducción](#introducción)
    - [Objetivos](#objetivos)
2. [Configuración del Entorno](#configuración-del-entorno)
    - [Requisitos](#requisitos)
    - [Variables de Entorno](#variables-de-entorno)
    - [Instalación](#instalación)
3. [Estructura del Proyecto](#estructura-del-proyecto)
    - [Arquitectura](#arquitectura)
    - [Estructura de Carpetas](#estructura-de-carpetas)
4. [Implementación de Pruebas de Rendimiento en JMeter](#implementación-de-pruebas-de-rendimiento-en-jmeter)
    - [Crear un Plan de Prueba](#crear-un-plan-de-prueba)
    - [Configurar el Servidor de Destino](#configurar-el-servidor-de-destino)
    - [Crear los Elementos de Prueba](#crear-los-elementos-de-prueba)
    - [Configurar las Aserciones](#configurar-las-aserciones)
    - [Ejecutar la Prueba de Rendimiento](#ejecutar-la-prueba-de-rendimiento)
    - [Analizar los Resultados](#analizar-los-resultados)
    - [Integrar las Pruebas de Rendimiento en CI/CD](#integrar-las-pruebas-de-rendimiento-en-ci-cd)
5. [Pipeline de Jenkins](#pipeline-de-jenkins)
    - [Descripción del Pipeline](#descripción-del-pipeline)
    - [Estructura del Pipeline](#estructura-del-pipeline)
    - [Resultados y Logs](#resultados-y-logs)
6. [Uso y Mantenimiento](#uso-y-mantenimiento)
    - [Actualización de Dependencias](#actualización-de-dependencias)
    - [Solución de Problemas](#solución-de-problemas)
7. [Conclusiones y Próximos Pasos](#conclusiones-y-próximos-pasos)
8. [Referencias](#referencias)
9. [Anexos](#anexos)
    - [Archivos de Configuración](#archivos-de-configuración)
    - [Scripts Utilizados](#scripts-utilizados)

---

## Introducción

Este proyecto utiliza **JMeter** y **Jenkins** para implementar pruebas de rendimiento en un ciclo de integración continua. Las pruebas están diseñadas para evaluar el rendimiento del sistema y detectar posibles cuellos de botella antes de la implementación en producción. Para estas pruebas, se utilizó la PokeAPI como servidor de destino para simular solicitudes HTTP y medir el rendimiento.

**Repositorio:** [JMeter-CI-CD](https://github.com/vngerus/JMeter-CI-CD)

### Objetivos
- Implementar pruebas de rendimiento utilizando JMeter.
- Integrar estas pruebas en un ciclo de integración continua mediante Jenkins.
- Asegurar que el sistema pueda manejar la carga esperada antes de su despliegue en producción.

## Configuración del Entorno

### Requisitos
- **Java JDK:** Versión 11 o superior.
- **JMeter:** Versión 5.6.3 o superior.
- **Jenkins:** Instalado y configurado para ejecutar pipelines.

### Variables de Entorno
- `JAVA_HOME`: Ruta al JDK de Java.
- `JMETER_HOME`: Ruta a la instalación de JMeter.

### Instalación
1. Clona el repositorio:
   ```bash
   git clone https://github.com/vngerus/JMeter-CI-CD.git
   ```
2. Navega al directorio del proyecto:
   ```bash
   cd JMeter-CI-CD
   ```
3. Configura JMeter y Java en tu entorno de desarrollo.

## Estructura del Proyecto

### Arquitectura
- **Pruebas de Rendimiento:** Las pruebas de rendimiento son gestionadas y ejecutadas desde JMeter.
- **Integración Continua:** Jenkins se encarga de ejecutar las pruebas automáticamente en cada cambio de código.

### Estructura de Carpetas
- `tests/`: Contiene los archivos .jmx utilizados para las pruebas de rendimiento.
- `reports/`: Carpeta donde se almacenan los reportes de JMeter.
- `scripts/`: Scripts personalizados para la ejecución de pruebas.

## Implementación de Pruebas de Rendimiento en JMeter

### Crear un Plan de Prueba
1. **Nuevo Plan:** Inicia JMeter y crea un nuevo Plan de Prueba (Test Plan).
2. **Configuraciones:** Define el número de hilos, duración de la prueba, solicitudes HTTP, etc.

### Configurar el Servidor de Destino
1. **Servidor de Destino:** Especifica el servidor donde se realizarán las pruebas (en este caso, la PokeAPI).
2. **Configuración:** Incluye la URL del servidor y otros parámetros de conexión.

### Crear los Elementos de Prueba
1. **Elementos de Prueba:** Crea controladores de muestra (Sampler) y controladores de lógica (Logic Controller) para definir la carga de trabajo y la lógica de la prueba.

### Configurar las Aserciones
1. **Aserciones:** Configura las aserciones para verificar los resultados, como tiempos de respuesta y éxito de las solicitudes.

### Ejecutar la Prueba de Rendimiento
1. **Ejecutar Prueba:** Ejecuta la prueba en JMeter y monitorea las estadísticas en tiempo real.

### Analizar los Resultados
1. **Análisis:** Utiliza los informes y gráficos generados por JMeter para identificar problemas de rendimiento.

### Integrar las Pruebas de Rendimiento en CI/CD
1. **Integración Continua:** Configura Jenkins para ejecutar las pruebas de rendimiento automáticamente en cada compilación o despliegue.

## Pipeline de Jenkins

### Descripción del Pipeline
El pipeline está configurado para ejecutar las pruebas de rendimiento, archivar los resultados, y notificar al equipo mediante Slack.

### Estructura del Pipeline
1. **Ejecución de Pruebas:** Ejecuta JMeter desde la línea de comandos para realizar las pruebas de rendimiento.
2. **Archivar Resultados:** Guarda los resultados de las pruebas en Jenkins.
3. **Publicar Reporte:** Publica los reportes de rendimiento generados por JMeter.
4. **Notificación a Slack:** Envía notificaciones al canal `#time-tracker-ci` en Slack.

```groovy
pipeline {
    agent any

    stages {
        stage('Run JMeter Tests') {
            steps {
                bat 'D:\\Archivos\\apache-jmeter-5.6.3\\bin\\jmeter.bat -n -t D:\\Archivos\\Respuesta.jmx -l D:\\Archivos\\resultados.jtl -e -o D:\\Archivos\\jmeter-report'
            }
        }
        
        stage('Archive Results') {
            steps {
                archiveArtifacts artifacts: 'D:\\Archivos\\jmeter-report/**/*', allowEmptyArchive: true
            }
        }
        
        stage('Publish Report') {
            steps {
                publishHTML([allowMissing: false,
                             alwaysLinkToLastBuild: true,
                             keepAll: true,
                             reportDir: 'D:\\Archivos\\jmeter-report',
                             reportFiles: 'index.html',
                             reportName: 'JMeter Performance Report'])
            }
        }
    }
    
    post {
        always {
            slackSend(channel: '#time-tracker-ci', message: "Pipeline en proceso: ${currentBuild.fullDisplayName}")
        }
        success {
            slackSend(channel: '#time-tracker-ci', message: "Éxito: El Pipeline se ha completado con éxito: ${currentBuild.fullDisplayName}")
        }
        failure {
            slackSend(channel: '#time-tracker-ci', message: "Fallo: El Pipeline ha fallado: ${currentBuild.fullDisplayName}")
        }
        cleanup {
            cleanWs()
        }
    }
}
```


### Resultados y Logs

Ejemplo de ejecución:

```
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in C:\ProgramData\Jenkins\.jenkins\workspace\Jmeter con Pipeline
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Run JMeter Tests)
[Pipeline] bat
C:\ProgramData\Jenkins\.jenkins\workspace\Jmeter con Pipeline>D:\Archivospache-jmeter-5.6.3in\jmeter.bat -n -t D:\Archivos\Respuesta.jmx -l D:\Archivos
esultados.jtl -e -o D:\Archivos\jmeter-report 
...
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Archive Results)
[Pipeline] archiveArtifacts
Guardando archivos
[Pipeline] }
[Pipeline] stage
[Pipeline] { (Publish Report)
[Pipeline] publishHTML
[htmlpublisher] Archiving HTML reports...
...
[Pipeline] End of Pipeline
Finished: SUCCESS
```

## Uso y Mantenimiento

### Actualización de Dependencias
Actualiza JMeter y las dependencias relevantes según sea necesario.

### Solución de Problemas
- **Errores de Configuración:** Verifica las rutas y configuraciones en Jenkins y JMeter.
- **Problemas con JMeter:** Asegúrate de que `JMETER_HOME` esté correctamente configurado.

## Conclusiones y Próximos Pasos

El pipeline automatiza las pruebas de rendimiento, asegurando un entorno limpio y eficiente para cada ejecución. Los próximos pasos incluyen la optimización de las pruebas y la mejora continua del proyecto.

## Referencias
- **JMeter Documentation:** https://jmeter.apache.org/usermanual/index.html
- **Jenkins Documentation:** https://www.jenkins.io/doc/
- **PokeAPI:** https://pokeapi.co/

## Anexos

### Archivos de Configuración
- `Respuesta.jmx`: Archivo de prueba de JMeter.
- `Jenkinsfile`: Archivo de configuración del pipeline en Jenkins.
