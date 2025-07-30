# Reto Técnico DevSecOps - Mibanco

Implementación completa de flujo DevSecOps para el reto técnico de Mibanco. Incluye aplicación Flask, pipelines CI/CD automatizados, escaneo de seguridad y despliegue en Azure Kubernetes Service.

**Funcionalidad**: API que responde `{"message": "Hola Mibanco"}` en el endpoint raíz.

## Arquitectura

- **Aplicación**: Python Flask containerizada
- **Registry**: Azure Container Registry (ACR)
- **Orchestration**: Azure Kubernetes Service (AKS)
- **Ingress**: NGINX Ingress Controller
- **CI/CD**: GitHub Actions
- **Security**: Dependency Review + Grype scanning

# Diagramas

## Arquitectura de Infraestructura

```mermaid
graph TB
    subgraph subGraph0["Ingress Namespace"]
            IC["NGINX Ingress Controller"]
            LB["Load Balancer"]
    end
    subgraph subGraph1["Mibanco App"]
            P1["Pod 1<br>mibanco-app<br> Single replica"]
    end
    subgraph subGraph2["Default Namespace"]
            subGraph1
            SVC["Service<br>ClusterIP"]
            HPA["HPA<br>1-10 replicas<br>📈 Can scale up"]
            ING["Ingress<br>mibanco-app"]
            P2["Pod 2<br>📈 On demand"]
            P3["Pod 3<br>📈 On demand"]
    end
    subgraph subGraph3["AKS Cluster"]
            subGraph0
            subGraph2
    end
    subgraph subGraph4["Resource Group"]
            subGraph3
            ACR["Azure Container Registry<br>acrmibancodevsecops*"]
    end
    subgraph subGraph5["Azure Cloud"]
            subGraph4
    end
    subgraph External["External"]
            USER["👤 User"]
            POSTMAN["📱 Postman"]
    end
    subgraph GitHub["GitHub"]
            REPO["📁 Repository"]
            ACTIONS["⚙️ GitHub Actions"]
            RELEASES["📦 Releases"]
    end
        USER --> LB
        POSTMAN --> LB
        LB --> IC
        IC --> ING
        ING --> SVC
        SVC --> P1
        HPA -.-> P1
        HPA -. Can create .-> P2 & P3
        ACTIONS --> RELEASES & ACR
        ACR --> P1

        style IC fill:#f3e5f5
        style LB fill:#f3e5f5
        style P1 fill:#e8f5e8
        style HPA fill:#fff3e0
        style P2 fill:#fff3e0,stroke-dasharray: 5 5
        style ACR fill:#e1f5fe
        style P3 fill:#fff3e0,stroke-dasharray: 5 5
```

## Flujo CI/CD Completo

```mermaid
graph TB
    subgraph "👨‍💻 Developer Workflow"
        DEV[Developer]
        BRANCH[Feature Branch]
        PR[Pull Request]
    end
    
    subgraph "🔒 CI Pipeline (Pull Request)"
        SEC[Security Workflow]
        DEP_REV[Dependency Review <br/> 📊 Python packages]
        TRIVY[Grype Scan <br/> 🔍 Container vulnerabilities]
        BUILD_TEST[Build Test <br/> 🔨 Docker build validation]
    end
    
    subgraph "🚀 CD Pipeline (Push to Main)"
        CD[CD Workflow]
        BUILD[Build Image <br/> 🏗️ Docker build]
        PUSH[Push to ACR <br/> 📤 Container registry]
        REL[Create Release <br/> 📦 GitHub release]
        DEPLOY[Deploy to AKS <br/> ☸️ Apply manifests]
        VALIDATE[Health Validation <br/> ✅ App testing]
    end
    
    subgraph "☁️ Azure Infrastructure"
        ACR_INFRA[Azure Container Registry]
        AKS_INFRA[AKS Cluster]
        PODS[Running Pod 🏃‍♂️ <br/> mibanco-app]
        LB_INFRA[Load Balancer <br/> 🌐 Public IP]
    end
    
    subgraph "🧪 Testing & Validation"
        HEALTH[Health Check ❤️ <br/> /health endpoint]
        APP_TEST[App Test 📝 <br/> 'Hola Mibanco']
        POSTMAN_TEST[Postman Testing 📱 <br/> Manual validation]
    end
    
    %% Developer flow
    DEV --> BRANCH
    BRANCH --> PR
    
    %% CI trigger
    PR --> SEC
    SEC --> DEP_REV
    SEC --> TRIVY
    SEC --> BUILD_TEST
    
    %% Merge condition
    DEP_REV --> MERGE{✅ All Checks Pass?}
    TRIVY --> MERGE
    BUILD_TEST --> MERGE
    
    %% CD trigger
    MERGE -->|Merge to Main| CD
    CD --> BUILD
    BUILD --> PUSH
    PUSH --> REL
    REL --> DEPLOY
    DEPLOY --> VALIDATE
    
    %% Infrastructure interaction
    PUSH --> ACR_INFRA
    DEPLOY --> AKS_INFRA
    AKS_INFRA --> PODS
    PODS --> LB_INFRA
    
    %% Testing flow
    VALIDATE --> HEALTH
    VALIDATE --> APP_TEST
    LB_INFRA --> POSTMAN_TEST
    
    %% Styling
    style SEC fill:#ffebee
    style CD fill:#e8f5e8
    style ACR_INFRA fill:#e1f5fe
    style AKS_INFRA fill:#e1f5fe
    style PODS fill:#f1f8e9
    style HEALTH fill:#e8f5e8
    style APP_TEST fill:#e8f5e8
    style POSTMAN_TEST fill:#fff3e0
```

## Workflow de Infraestructura (Manual)

```mermaid
graph LR
    subgraph "🎯 Manual Trigger"
        ADMIN[👨‍💻 Admin]
        DISPATCH[Workflow Dispatch<br/>🔘 Manual trigger]
    end
    
    subgraph "🏗️ Infrastructure Workflow"
        INFRA[Infrastructure Action]
        CREATE{Action Type}
        
        subgraph "📦 Create Resources"
            CHECK_RG[Check Resource Group<br/>📂 Verify existing RG]
            CREATE_ACR[Create ACR<br/>🐳 Container registry]
            CREATE_AKS[Create AKS<br/>☸️ Kubernetes cluster]
            CONFIG_ACR[Configure ACR Access<br/>🔐 Admin credentials]
            INSTALL_NGINX[Install NGINX Ingress<br/>🌐 Load balancer]
        end
        
        subgraph "🗑️ Destroy Resources"
            DELETE_AKS[Delete AKS<br/>💥 Remove cluster]
            DELETE_ACR[Delete ACR<br/>💥 Remove registry]
            PRESERVE_RG[Preserve RG<br/>✅ Keep existing resources]
        end
    end
    
    subgraph "☁️ Azure Resources"
        RG[Resource Group<br/>📁 Existing]
        ACR_RES[ACR Registry<br/>🐳 acrmibancodevsecops*]
        AKS_RES[AKS Cluster<br/>☸️ aks-mibanco-cluster]
        NGINX_RES[NGINX Ingress<br/>🌐 Load balancer service]
    end
    
    %% Flow
    ADMIN --> DISPATCH
    DISPATCH --> INFRA
    INFRA --> CREATE
    
    %% Create flow
    CREATE -->|create| CHECK_RG
    CHECK_RG --> CREATE_ACR
    CREATE_ACR --> CREATE_AKS
    CREATE_AKS --> CONFIG_ACR
    CONFIG_ACR --> INSTALL_NGINX
    
    %% Destroy flow
    CREATE -->|destroy| DELETE_AKS
    DELETE_AKS --> DELETE_ACR
    DELETE_ACR --> PRESERVE_RG
    
    %% Resources
    CHECK_RG --> RG
    CREATE_ACR --> ACR_RES
    CREATE_AKS --> AKS_RES
    INSTALL_NGINX --> NGINX_RES
    
    %% Cleanup
    DELETE_AKS -.-> AKS_RES
    DELETE_ACR -.-> ACR_RES
    
    style CREATE_ACR fill:#e1f5fe
    style CREATE_AKS fill:#e1f5fe
    style DELETE_AKS fill:#ffebee
    style DELETE_ACR fill:#ffebee
    style RG fill:#f3e5f5
    style PRESERVE_RG fill:#e8f5e8
```

## 📁 Estructura del Proyecto

```
├── .github/workflows/
│   ├── ci.yml              # Escaneo de seguridad en PRs (check requireds)
│   ├── cd.yml              # Pipeline de deployment
│   └── infrastructure.yml  # Provisioning de infraestructura
├── src/
│   ├── app.py              # Aplicación Flask
│   ├── requirements.txt    # Dependencias Python
│   └── Dockerfile         # Imagen de contenedor
├── k8s/
│   ├── deployment.yml     # Pods de la aplicación
│   ├── service.yml        # Servicio interno
│   ├── hpa.yml            # Auto-scaling
│   └── ingress.yml        # Acceso externo
└── README.md
```

## ⚙️ Setup e Instalación

### 1. **Configurar Secretos en GitHub**

```bash
# Settings → Secrets → Actions
AZURE_CREDENTIALS = {
  "clientId": "tu-client-id",
  "clientSecret": "tu-client-secret", 
  "subscriptionId": "tu-subscription-id",
  "tenantId": "tu-tenant-id"
}
```

### 2. **Crear Infraestructura**

1. Actions → **Provision Infrastructure** 
2. Run workflow → Action: `create`
3. Copiar ACR name del output

### 3. **Configurar CD Pipeline**

Actualizar `REGISTRY_NAME` en `.github/workflows/cd.yml` con el ACR creado.

### 4. **Activar Branch Protection**

Settings → Rules → New Ruleset:
- Require pull request reviews (1)
- Restrict deletions
- Block foce pushes

## Workflows

### **CI** (Pull Requests)
- **Dependency Review**: Análisis de vulnerabilidades en dependencias
- **Grype Scan**: Escaneo de vulnerabilidades en contenedores

### **CD** (Push a main)
1. **Build**: Construye y sube imagen a ACR
2. **Release**: Crea GitHub release automático
3. **Deploy**: Despliega a AKS con todos los manifiestos
4. **Validate**: Verifica health de la aplicación

### **Infrastructure** (Manual)
- `create`: Provisiona AKS, ACR e Ingress
- `destroy`: Elimina recursos

## Testing

### **Obtener IP Externa**
```bash
kubectl get service -n ingress-nginx ingress-nginx-controller
```

### **Test de la Aplicación**
```bash
# Main endpoint
GET http://[EXTERNAL-IP]/
Response: {"message": "Hola Mibanco"}

# Health check
GET http://[EXTERNAL-IP]/health  
Response: {"status": "healthy"}
```

## Seguridad Implementada

- ✅ **Dependency scanning** en PRs
- ✅ **Container vulnerability scanning** con Grype
- ✅ **Non-root containers** con security contexts
- ✅ **Rulesets** con PR approvals
- ✅ **Resource limits** y security policies

## 📊 Manifiestos K8s

- **deployment.yml**: 1 replica con health checks y security contexts
- **service.yml**: ClusterIP para comunicación interna
- **hpa.yml**: Auto-scaling 2-10 pods basado en CPU/Memory
- **ingress.yml**: NGINX para acceso externo

## Comandos Útiles

```bash
# Conectar al cluster
az aks get-credentials --resource-group [RG] --name aks-mibanco-cluster-prd

# Ver pods
kubectl get pods -l app=mibanco-app

# Ver logs
kubectl logs -l app=mibanco-app

# Port forward para test local
kubectl port-forward service/mibanco-app-service 8080:80
```

## ✅ Requisitos Cumplidos

### **Funcionales**
- [x] Aplicación "Hola Mibanco"
- [x] Automatización con GitHub Actions  
- [x] Trunk-based development
- [x] Recursos Azure automatizados

### **Técnicos**
- [x] AKS + ACR + Ingress Controller
- [x] Build y push automatizado
- [x] Deploy con manifiestos K8s completos
- [x] Validación de pods y servicios
- [x] Rulesets con PR approvals
- [x] Escaneo de seguridad (opcional)

## 🎯 Características Destacadas

- **GitOps**: Trunk-based development con PR obligatorios
- **Security**: Scanning automatizado en pipeline
- **HA**: 3 replicas con auto-scaling
- **Monitoring**: Health checks y resource monitoring
- **Automation**: Infrastructure as Code completo