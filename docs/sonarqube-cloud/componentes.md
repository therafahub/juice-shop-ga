# Componentes del escaneo con SonarQube Cloud

```mermaid
flowchart TB
  Dev["Desarrollador"]

  subgraph GitHub["GitHub"]
    Repo["Repositorio<br/>código y workflow"]
    Event["Eventos<br/>push / pull_request / workflow_dispatch"]
    Actions["GitHub Actions<br/>orquesta el job"]
    Secret["Secret SONAR_TOKEN"]
    Repo --> Event --> Actions
    Secret -. "inyectado al job" .-> Actions
  end

  subgraph Runner["Runner hospedado y temporal"]
    Checkout["actions/checkout<br/>descarga el repositorio"]
    ScanAction["SonarQube scan action"]
    Scanner["SonarScanner<br/>analiza las fuentes"]
    Checkout --> ScanAction --> Scanner
  end

  subgraph Sonar["SonarQube Cloud"]
    Endpoint["Autenticación y endpoint<br/>sonarcloud.io"]
    Engine["Procesador del análisis"]
    Project["Proyecto Sonar<br/>hallazgos, métricas y Quality Gate"]
    Endpoint --> Engine --> Project
  end

  Dev -->|"push / PR"| Repo
  Actions -->|"asigna runner"| Checkout
  Checkout <-->|"HTTPS / TCP 443<br/>checkout y acciones"| Repo
  Scanner -->|"HTTPS / TCP 443<br/>informe de análisis"| Endpoint
  Dev -->|"HTTPS / TCP 443"| Project
```

## Responsabilidad de cada componente

| Componente | Función |
|---|---|
| GitHub Repository | Guarda el código y `.github/workflows/curso_gha-sonar.yml`. |
| GitHub Actions | Inicia el job según el evento y asigna el runner. |
| `SONAR_TOKEN` | Permite autenticar el análisis; se configura como secreto del repositorio. |
| Runner | Descarga el checkout y las acciones; es efímero y hospedado por GitHub. |
| SonarQube scan action / SonarScanner | Analiza los archivos y publica el informe. No ejecuta el build ni las pruebas en la configuración actual. |
| SonarQube Cloud | Procesa el informe y presenta los hallazgos, métricas y Quality Gate. |

El análisis se asocia a la organización y al Project Key configurados como
argumentos del scanner en el workflow. Esos identificadores deben coincidir
exactamente con los del proyecto creado en SonarQube Cloud.
