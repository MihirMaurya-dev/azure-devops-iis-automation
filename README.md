<div align="center">

# ⚡ Azure DevOps Self-Hosted Agent IIS Automation
### Automated .NET Core Testing, Cobertura Code Coverage & Zero-Downtime IIS Site Deployment

[![Platform](https://img.shields.io/badge/Orchestrator-Azure_DevOps_Pipelines-0078D7?style=for-the-badge&logo=azuredevops&logoColor=white)](https://dev.azure.com)
[![Testing](https://img.shields.io/badge/Unit_Testing-.NET_Core_CLI-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)](https://dotnet.microsoft.com)
[![Coverage](https://img.shields.io/badge/Code_Coverage-Cobertura_XML-00FF66?style=for-the-badge&logo=codecov&logoColor=0B0813)](#-pipeline-lifecycle)
[![Web Server](https://img.shields.io/badge/Web_Server-Microsoft_IIS_appcmd-00E5FF?style=for-the-badge&logo=windows&logoColor=0B0813)](#-automated-iis-deployment)

**Architected by [Mihir Maurya](https://github.com/MihirMaurya-dev) · Enterprise Cloud Architect & DevSecOps**

</div>

---

## 🏗️ Architectural Lifecycle

This repository demonstrates a **production CI/CD automation pipeline** executing on a dedicated self-hosted Windows build agent (`Velociraptor-Agent`). It automatically compiles .NET Core application source code, executes unit tests with XPlat code coverage collection, publishes dashboard metrics, and provisions an IIS web application dynamically via `appcmd.exe`.

```mermaid
flowchart LR
    subgraph S1["🧪 Step 1: Automated Unit Tests"]
        A[Git Push master] --> B[DotNetCoreCLI@2 test]
        B --> C[Collect XPlat Code Coverage]
    end

    subgraph S2["📊 Step 2: Quality Dashboard"]
        C --> D[PublishCodeCoverageResults@2]
        D --> E[Render Cobertura Dashboard in ADO]
    end

    subgraph S3["🚀 Step 3: Zero-Touch IIS Deployment"]
        E --> F[PowerShell@2 Task]
        F --> G[Ensure C:\inetpub\wwwroot\SimpleApp]
        G --> H[appcmd.exe add app /site.name:Default Web Site]
        H --> I[Live Production Web App]
    end

    classDef adoBlue fill:#08182b,stroke:#0078d7,stroke-width:2px,color:#fff;
    classDef testGreen fill:#032111,stroke:#00ff66,stroke-width:2px,color:#fff;
    classDef iisPurple fill:#1d0738,stroke:#9d4edd,stroke-width:2px,color:#fff;

    class S1,A,B,C adoBlue;
    class S2,D,E testGreen;
    class S3,F,G,H,I iisPurple;
```

---

## 📂 Repository Topology

```text
azure-devops-iis-automation/
├── azure-pipelines.yml       # 🔄 Azure DevOps Pipeline definition
├── Practice.slnx             # 📦 Solution file linking application & test projects
├── MathApp/                  # 💻 Core .NET Core application source code
│   ├── Class1.cs
│   └── MathApp.csproj
├── MathApp.Tests/            # 🧪 Unit test project using xUnit / MSTest
│   ├── UnitTest1.cs
│   └── MathApp.Tests.csproj
└── SimpleApp/                # 🌐 Static frontend deployed to IIS
    └── index.html
```

---

## ⚡ Pipeline Implementation Breakdown

### 1. Unit Testing & Coverage Collection
```yaml
- task: DotNetCoreCLI@2
  displayName: 'Run Unit Tests'
  inputs:
    command: 'test'
    projects: '**/*.csproj'
    arguments: '--collect "XPlat Code Coverage"'
    publishTestResults: true

- task: PublishCodeCoverageResults@2
  displayName: 'Publish Code Coverage Dashboard'
  inputs:
    summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
```

### 2. Idempotent IIS Web Application Configuration
```powershell
$targetPath = "C:\inetpub\wwwroot\SimpleApp"
if (-not (Test-Path $targetPath)) { New-Item -ItemType Directory -Force -Path $targetPath }
Copy-Item -Path "$(System.DefaultWorkingDirectory)\TestingPractice\SimpleApp\*" -Destination $targetPath -Recurse -Force

$appcmd = "C:\Windows\System32\inetsrv\appcmd.exe"
$appExists = & $appcmd list app /app.name:"Default Web Site/SimpleApp"
if (-not $appExists) {
    & $appcmd add app /site.name:"Default Web Site" /path:"/SimpleApp" /physicalPath:$targetPath /applicationPool:"DefaultAppPool"
}
```

---

<div align="center">

**[Portfolio](https://mihirmaurya-dev.github.io/)** · **[LinkedIn](https://linkedin.com/in/mihirmauryaa1)** · **[DevOps Knowledge Vault](https://github.com/MihirMaurya-dev/DevOps-Notes)**

</div>
