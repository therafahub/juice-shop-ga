# Secuencia del escaneo con SonarQube Cloud

```mermaid
sequenceDiagram
  actor Dev as Desarrollador
  participant GH as GitHub (repositorio y Actions)
  participant Runner as Runner temporal ubuntu-latest
  participant Scanner as SonarScanner
  participant SC as SonarQube Cloud

  Dev->>GH: Push, pull request o ejecución manual
  GH->>Runner: Inicia el job en el runner
  Runner->>GH: Checkout por HTTPS / TCP 443
  GH-->>Runner: Workflow y archivos del repositorio
  Runner->>GH: Descarga de acciones por HTTPS / TCP 443
  GH-->>Runner: SonarQube scan action
  Runner->>Scanner: Ejecuta acción con SONAR_TOKEN
  Scanner->>SC: Solicita configuración/reglas por HTTPS / TCP 443
  SC-->>Scanner: Configuración y reglas del proyecto
  Scanner->>Scanner: Analiza las fuentes seleccionadas
  Note over Scanner: Este workflow no hace build ni ejecuta tests
  Scanner->>SC: Envía informe de análisis por HTTPS / TCP 443
  Note over Scanner,SC: El informe puede incluir fuente para mostrar contexto de hallazgos
  SC->>SC: Procesa y guarda el análisis
  SC-->>Scanner: Estado del procesamiento
  Scanner-->>Runner: Resultado del paso de análisis
  Runner-->>GH: Estado del job y logs
  Dev->>SC: Consulta el proyecto desde el navegador por HTTPS / TCP 443
  SC-->>Dev: Issues, métricas y Quality Gate
```

## Conexiones

El tráfico indicado es iniciado por el runner o por el navegador del usuario.
Las respuestas regresan por las conexiones HTTPS existentes, por lo que este
flujo no requiere publicar el runner en Internet ni abrir un puerto entrante.
