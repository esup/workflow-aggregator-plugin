# Architecture Diagrams

## 1. Aggregator Pattern Structure

```
┌────────────────────────────────────────────────────────────────┐
│                    workflow-aggregator                         │
│                  (Meta/Bundle Plugin)                          │
│                                                                │
│  Purpose: Dependency Management & Distribution                │
│  Code: Minimal (only POM + metadata)                         │
└────────────────────┬───────────────────────────────────────────┘
                     │
                     │ Aggregates & Distributes
                     │
         ┌───────────┴───────────┬──────────────┬──────────────┐
         │                       │              │              │
         ▼                       ▼              ▼              ▼
┌──────────────────┐   ┌──────────────────┐   ┌─────────┐   ┌──────────┐
│ workflow-        │   │ workflow-        │   │pipeline-│   │pipeline- │
│ durable-task-    │   │ cps              │   │groovy-  │   │model-    │
│ step             │   │ (Groovy Engine)  │   │lib      │   │definition│
│                  │   │                  │   │         │   │          │
│ Durable Tasks    │   │ CPS Execution    │   │ Shared  │   │Declarative│
│                  │   │                  │   │Libraries│   │Syntax     │
└──────────────────┘   └──────────────────┘   └─────────┘   └──────────┘

         │                       │              │              │
         ▼                       ▼              ▼              ▼
┌──────────────────┐   ┌──────────────────┐   ┌─────────┐   ┌──────────┐
│ workflow-job     │   │ workflow-basic-  │   │pipeline-│   │pipeline- │
│                  │   │ steps            │   │build-   │   │input-    │
│ Job Type         │   │                  │   │step     │   │step      │
│                  │   │ Basic Steps      │   │         │   │          │
└──────────────────┘   └──────────────────┘   └─────────┘   └──────────┘

         │                       │              │
         ▼                       ▼              ▼
┌──────────────────┐   ┌──────────────────┐   ┌─────────┐
│ workflow-        │   │ pipeline-stage-  │   │pipeline-│
│ multibranch      │   │ step             │   │milestone│
│                  │   │                  │   │-step    │
│ Multi-branch     │   │ Stage Definition │   │         │
│ Support          │   │                  │   │Milestone│
└──────────────────┘   └──────────────────┘   └─────────┘
```

## 2. User Installation Flow

```
┌─────────────┐
│   User      │
│             │
└──────┬──────┘
       │
       │ 1. Install workflow-aggregator
       ▼
┌────────────────────────────────────┐
│  Jenkins Plugin Manager            │
│                                    │
│  Resolves Dependencies             │
└──────┬─────────────────────────────┘
       │
       │ 2. Auto-install all dependencies
       ▼
┌────────────────────────────────────┐
│  11 Core Pipeline Plugins          │
│  - workflow-durable-task-step      │
│  - workflow-cps                    │
│  - pipeline-groovy-lib             │
│  - workflow-job                    │
│  - workflow-basic-steps            │
│  - workflow-multibranch            │
│  - pipeline-build-step             │
│  - pipeline-input-step             │
│  - pipeline-stage-step             │
│  - pipeline-milestone-step         │
│  - pipeline-model-definition       │
└──────┬─────────────────────────────┘
       │
       │ 3. Restart Jenkins
       ▼
┌────────────────────────────────────┐
│  Complete Pipeline Functionality   │
│  Available                         │
└────────────────────────────────────┘
```

## 3. Developer Dependency Guidelines

### ❌ Wrong Approach
```
┌─────────────────────┐
│  Your Plugin        │
│                     │
│  depends on         │
│  workflow-aggregator│  ← Brings unnecessary dependencies
│                     │
└─────────────────────┘
         │
         │ Pulls in ALL 11 plugins
         ▼
┌─────────────────────┐
│  Heavy Dependencies │
│  (11 plugins)       │
│  - Bloated          │
│  - Slower builds    │
│  - Version conflicts│
└─────────────────────┘
```

### ✅ Correct Approach
```
┌─────────────────────┐
│  Your Plugin        │
│                     │
│  depends on         │
│  workflow-step-api  │  ← Only what you need
│                     │
└─────────────────────┘
         │
         │ Pulls in minimal API
         ▼
┌─────────────────────┐
│  Light Dependencies │
│  (1 API plugin)     │
│  - Fast             │
│  - Clean            │
│  - Maintainable     │
└─────────────────────┘
```

## 4. Component Interaction

```
┌─────────────────────────────────────────────────────────────┐
│                      Jenkins Core                           │
└────────┬────────────────────────────────────────────────────┘
         │
         │ Plugin Loading
         ▼
┌─────────────────────────────────────────────────────────────┐
│                  workflow-aggregator                        │
│  (Loaded but provides no direct functionality)             │
└────────┬────────────────────────────────────────────────────┘
         │
         │ Dependencies Loaded
         ▼
┌─────────────────────────────────────────────────────────────┐
│              Core Pipeline Components                       │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ Execution    │  │ DSL Parser   │  │ Job Types    │    │
│  │ Engine       │  │ (Groovy)     │  │              │    │
│  │ (CPS)        │  │              │  │              │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ Steps        │  │ Multi-branch │  │ Shared Libs  │    │
│  │ (Basic,      │  │ Support      │  │              │    │
│  │  Build,      │  │              │  │              │    │
│  │  Input,      │  │              │  │              │    │
│  │  Stage,      │  │              │  │              │    │
│  │  Milestone)  │  │              │  │              │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────┘
         │
         │ Provides
         ▼
┌─────────────────────────────────────────────────────────────┐
│              Pipeline Functionality to Users                │
│  - Scripted Pipeline                                        │
│  - Declarative Pipeline                                     │
│  - Shared Libraries                                         │
│  - Multi-branch Pipelines                                   │
└─────────────────────────────────────────────────────────────┘
```

## 5. Architecture Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    Layer 4: User Layer                      │
│  - Pipeline Scripts (Jenkinsfile)                          │
│  - Shared Libraries                                         │
│  - Job Configuration                                        │
└────────┬────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│              Layer 3: Aggregator Layer                      │
│  - workflow-aggregator (Meta Plugin)                       │
│  - Dependency Management                                    │
│  - Version Coordination                                     │
└────────┬────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│            Layer 2: Functional Plugin Layer                 │
│  - workflow-cps (Execution Engine)                         │
│  - workflow-job (Job Type)                                 │
│  - workflow-basic-steps (Steps)                            │
│  - pipeline-model-definition (Declarative Syntax)          │
│  - pipeline-groovy-lib (Shared Libraries)                  │
│  - workflow-multibranch (Multi-branch)                     │
│  - etc.                                                     │
└────────┬────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│               Layer 1: Jenkins Core Layer                   │
│  - Plugin Management                                        │
│  - Execution Framework                                      │
│  - UI Framework                                             │
│  - Security                                                 │
└─────────────────────────────────────────────────────────────┘
```

## 6. Plugin Dependency Graph (Maven)

```
                    workflow-aggregator (pom.xml)
                            │
                ┌───────────┴───────────┐
                │                       │
                │  dependencyManagement │
                │  (Jenkins BOM)        │
                │                       │
                └───────────┬───────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
    <dependencies>      (Managed        (Compatible
     11 plugins         Versions)       Versions)
```

## 7. Build and Release Flow

```
┌──────────────┐
│ Developer    │
│ commits code │
└──────┬───────┘
       │
       │ Push to GitHub
       ▼
┌────────────────────────┐
│  GitHub Repository     │
│  jenkinsci/workflow-   │
│  aggregator-plugin     │
└──────┬─────────────────┘
       │
       │ Triggers
       ▼
┌────────────────────────┐
│  Jenkinsfile           │
│  (Self-CI/CD)          │
│                        │
│  buildPlugin(          │
│    linux/JDK25        │
│    windows/JDK21      │
│  )                     │
└──────┬─────────────────┘
       │
       │ Build & Test
       ▼
┌────────────────────────┐
│  Maven Build           │
│  - Compile (none)      │
│  - Package HPI         │
│  - Run Tests (none)    │
└──────┬─────────────────┘
       │
       │ On success
       ▼
┌────────────────────────┐
│  Release to            │
│  Jenkins Update Center │
└────────────────────────┘
```

## Key Insights

### Aggregator Pattern Benefits
1. **Simplicity**: Minimal code, maximum value
2. **Consistency**: Guaranteed compatible versions
3. **Convenience**: One-click installation
4. **Maintainability**: Easy to update dependencies

### Design Trade-offs
- **Pro**: Easy for end users
- **Con**: Not suitable for plugin developers
- **Solution**: Clear documentation separating user vs developer use cases
