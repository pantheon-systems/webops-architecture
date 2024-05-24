# Mid-level complexity workflows

## Custom Upstream Setup

In this setup, the company has

- A custom upstream that's based on the Pantheon default WordPress upstream
- The custom upstream has their standard custom theme and suite of plugins that all sites use
- The canary site uses the custom upstream "as-is". It's used for testing changes to the upstream before releasing it to the rest of the sites.
- Each site uses the theme and plugins from the upstream. They can also include a child theme and other plugins that they specifically need.
- A core developer team maintains the upstream and common codebase.
- Individual sites can be maintained by more developers that should have more limited access.

Useful for

- Teams with several developers
- Teams with developers at varying skillsets
- Teams that want to restrict some of their developers to a subset of the sites maintained
- Sites all share a common codebase but individual sites maintain their own plugins and theme.

@todo

- Canary diagram using multidevs on sites that are tagged with canary tag

### Custom Upstream Diagrams

```mermaid
---
title: Dependency chart
---
graph
    UpstreamPantheon["Pantheon Upstream"]:::repo

    subgraph github [GitHub]
        UpstreamCustom["Custom Upstream\nWith custom theme, plugins"]:::repo
    end

    subgraph pantheon [Pantheon]
        subgraph site0 [Canary Site]
            direction LR
            EnvDev0(Multidev Environment):::env
        end

        subgraph site1 [Site 1]
            direction LR
            EnvDev1(Dev Environment):::env
            EnvTest1(Test Environment):::env
            EnvLive1(Live Environment):::env
        end

        subgraph site2 [Site 2]
            direction LR
            EnvDev2(Dev Environment):::env
            EnvTest2(Test Environment):::env
            EnvLive2(Live Environment):::env
        end

        subgraph site3 [Site 3]
            direction LR
            EnvDev3(Dev Environment):::env
            EnvTest3(Test Environment):::env
            EnvLive3(Live Environment):::env
        end
    end

    %% Links
    
    UpstreamPantheon --> UpstreamCustom

    UpstreamCustom -- Feature branch --> EnvDev0
    UpstreamCustom -- Master branch --> EnvDev1
    UpstreamCustom -- Master Branch --> EnvDev2
    UpstreamCustom -- Master Branch --> EnvDev3

    EnvDev1 --> EnvTest1
    EnvTest1 --> EnvLive1

    EnvDev2 --> EnvTest2
    EnvTest2 --> EnvLive2

    EnvDev3 --> EnvTest3
    EnvTest3 --> EnvLive3

    %% Styles
    style github fill:#4078c0,color:#FFF
    style pantheon fill:#181D21,color:#FFF
    style site0 fill:#78868E,color:#000
    style site1 fill:#78868E,color:#000
    style site2 fill:#78868E,color:#000
    style site3 fill:#78868E,color:#000
    classDef env fill:#EFD01B
    classDef repo fill:#fafafa,color:#000

```

```mermaid
---
title: Code deployment activity diagram
---
sequenceDiagram

    % People & Systems
    actor Dev as Developer
    participant PU as Pantheon Upstream
    participant CU as Custom Upstream
    participant S0 as Canary Site
    participant S1 as Site 1
    participant S2 as Site 2

    % Actions

    rect rgb(184,192,64)
        Note over Dev,CU: General Development Phase
        PU->>CU: WP Core update
        Dev->>Dev: Code new features in local
        Dev->>CU: Dev creates a feature for all sites
    end

    rect rgb(184,192,64)
        Note over Dev,S2: Custom Development Phase
        Dev->>Dev: Code new features in local
        Dev->>S2: Dev creates a feature just for Site 2
    end

    rect rgb(64,120,192)
        Note over CU,S0: Testing Phase
        CU->>+S0: Deploy for testing
        S0->>-CU: Testing passed
    end
    rect rgb(192,72,64)
        Note over CU,S2: Deployment Phase
        par Deploying to Site 1
            CU->>S1: Deploy to Site 1
            Note over S1: Site 1 goes through its deployment process
        and Deploying to Site 2
            CU->>S2: Deploy to Site 2
            Note over S2: Site 2 goes through its deployment process
        end
    end

```

### Example Custom Upstream Code Structure

| Plugin / Theme | Managed Source |
|----------------|----------------|
| All-in-one SEO | Upstream       |
| Autoptimize    | Upstream       |
| Wordfence      | Upstream       |
| Custom Forms Plugin | Upstream  |
| Custom Parent Theme | Upstream  |
| Redirection    | Pantheon Site  |
| Custom Menu Plugin | Pantheon Site |
| Custom Child Theme | Pantheon Site |

### Custom Upstream User Permission Scope

| User Group | Primary Activity | System Access |
|------------|------------------|---------------|
| Senior Developers | Maintain overall strategy | Upstream Repo |
| Web Developers | Maintain all site content | Pantheon Deployments |
| Contractors | Maintain specific site content | Pantheon Development |

## Monorepo Setup

In this setup, the company has

- A custom upstream that's based on the Pantheon default WordPress upstream
- The custom upstream has all of the code for every site.
- The canary site uses the custom upstream "as-is". It's used for testing changes to the upstream before releasing it to the rest of the sites.
- Each site gets the full codebase deployed to it. They activate only the plugins and theme that it particularly needs to use.
- A CI script is often used to deploy the changes to all the sites en mass.

Useful for

- Teams with a few developers
- The team maintains a very large number of sites that run a very similar codebase

### Monorepo Diagrams

```mermaid
---
title: Dependency chart
---
graph
    UpstreamPantheon["Pantheon Upstream"]:::repo

    subgraph github [GitHub]
        UpstreamCustom["Custom Upstream\nWith custom themes, all plugins"]:::repo
    end

    subgraph pantheon [Pantheon]
        subgraph site0 [Canary Site]
            direction LR
            EnvDev0(Multidev Environment):::env
        end

        subgraph site1 [Site 1]
            direction LR
            EnvDev1(Dev Environment):::env
            EnvTest1(Test Environment):::env
            EnvLive1(Live Environment):::env
        end

        subgraph site2 [Site 2]
            direction LR
            EnvDev2(Dev Environment):::env
            EnvTest2(Test Environment):::env
            EnvLive2(Live Environment):::env
        end

        subgraph site3 [Site 3]
            direction LR
            EnvDev3(Dev Environment):::env
            EnvTest3(Test Environment):::env
            EnvLive3(Live Environment):::env
        end
    end

    %% Links
    
    UpstreamPantheon --> UpstreamCustom

    UpstreamCustom -- Feature branch --> EnvDev0
    UpstreamCustom -- Master branch --> EnvDev1
    UpstreamCustom -- Master Branch --> EnvDev2
    UpstreamCustom -- Master Branch --> EnvDev3

    EnvDev1 --> EnvTest1
    EnvTest1 --> EnvLive1

    EnvDev2 --> EnvTest2
    EnvTest2 --> EnvLive2

    EnvDev3 --> EnvTest3
    EnvTest3 --> EnvLive3

    %% Styles
    style github fill:#4078c0,color:#FFF
    style pantheon fill:#181D21,color:#FFF
    style site0 fill:#78868E,color:#000
    style site1 fill:#78868E,color:#000
    style site2 fill:#78868E,color:#000
    style site3 fill:#78868E,color:#000
    classDef env fill:#EFD01B
    classDef repo fill:#fafafa,color:#000

```

```mermaid
---
title: Code deployment activity diagram
---
sequenceDiagram

    % People & Systems
    actor Dev as Developer
    participant PU as Pantheon Upstream
    participant CU as Custom Upstream
    participant S0 as Canary Site
    participant S1 as Site 1
    participant S2 as Site 2

    % Actions

    rect rgb(184,192,64)
        Note over Dev,CU: New Feature for all sites
        PU->>CU: WP Core update
        Dev->>Dev: Code new features in local
        Dev->>CU: Dev creates a feature for all sites

        rect rgb(64,120,192)
            Note over CU,S0: Testing Phase
            CU->>+S0: Deploy for testing
            S0->>-CU: Testing passed
        end

        rect rgb(192,72,64)
            Note over CU,S2: Deployment Phase
            par CU to S1
                CU->>S1: Deploy to Site 1
                Note over S1: Site 1 goes through its deployment process
            and CU to S2
                CU->>S2: Deploy to Site 2
                Note over S2: Site 2 goes through its deployment process
            end
        end
    end

    rect rgb(184,192,64)
        Note over Dev,S2: New Feature for one site
        Dev->>Dev: Code new features in local
        Dev->>CU: Dev creates a feature just for Site 1. 

        rect rgb(192,72,64)
            Note over CU,S2: Deployment Phase
            par CU to S1
                CU->>S1: Deploy to Site 1
                Note over S1: Site 1 goes through its deployment process. Site 1 uses the change.
            and CU to S2
                CU->>S2: Deploy to Site 2
                Note over S2: Site 2 goes through its deployment process. Site 2 ignores the code.
            end
        end
    end
```

### Example Monorepo Code Structure

| Plugin / Theme | Managed Source | Used By   |
|----------------|----------------|-----------|
| All-in-one SEO | Upstream       | All sites |
| Autoptimize    | Upstream       | All sites |
| Wordfence      | Upstream       | All sites |
| Redirection    | Upstream       | One site  |
| Custom Forms Plugin | Upstream  | All sites |
| Custom Menu Plugin  | Upstream  | One site  |
| Custom Parent Theme | Upstream  | All sites |
| Custom Child Themes | Upstream  | One site  |
