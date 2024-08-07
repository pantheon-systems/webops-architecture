# Workflows that use custom upstreams

## Minimal Custom Upstream

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

### Dependencies in a minimal setup

```mermaid
---
title: Dependency chart
---
graph
    UpstreamPantheon["Pantheon Upstream"]:::repo

    subgraph github [GitHub]
        UpstreamCustom["Custom Upstream\nWith parent theme, custom plugins"]:::repo
    end

    subgraph pantheon [Pantheon]
        subgraph site0 [Canary Site]
            direction LR
            EnvMD0(Multidev Environment):::env
            EnvDev0(Dev Environment):::env
        end

        subgraph site1 [Site 1]
            direction LR
            SitePlugins1(Site-specific plugins):::note
            SiteTheme1(Child theme):::note
            EnvDev1(Dev Environment):::env
            EnvTest1(Test Environment):::env
            EnvLive1(Live Environment):::env
        end

        subgraph site2 [Site 2]
            direction LR
            SitePlugins2(Site-specific plugins):::note
            SiteTheme2(Child theme):::note
            EnvDev2(Dev Environment):::env
            EnvTest2(Test Environment):::env
            EnvLive2(Live Environment):::env
        end

        subgraph site3 [Site 3]
            direction LR
            SitePlugins3(Site-specific plugins):::note
            SiteTheme3(Child theme):::note
            EnvDev3(Dev Environment):::env
            EnvTest3(Test Environment):::env
            EnvLive3(Live Environment):::env
        end
    end

    %% Links
    
    UpstreamPantheon --> UpstreamCustom

    UpstreamCustom -- Feature branch --> EnvMD0
    UpstreamCustom -- Canary branch --> EnvDev0
    UpstreamCustom -- Master branch --> EnvDev1
    UpstreamCustom -- Master Branch --> EnvDev2
    UpstreamCustom -- Master Branch --> EnvDev3

    SitePlugins1 ~~~ SiteTheme1
    SitePlugins2 ~~~ SiteTheme2
    SitePlugins3 ~~~ SiteTheme3

    EnvDev1 --> EnvTest1
    EnvTest1 --> EnvLive1

    EnvDev2 --> EnvTest2
    EnvTest2 --> EnvLive2

    EnvDev3 --> EnvTest3
    EnvTest3 --> EnvLive3

    %% Styles
    style github fill:#4078c0,color:#FFF
    style pantheon fill:#181D21,color:#FFF
    style note fill:#CCC,color:#000
    style site0 fill:#78868E,color:#000
    style site1 fill:#78868E,color:#000
    style site2 fill:#78868E,color:#000
    style site3 fill:#78868E,color:#000
    classDef env fill:#EFD01B
    classDef repo fill:#fafafa,color:#000

```

### Processes for deploying code changes in a minimal setup

```mermaid
---
title: Deploying a new feature to all sites
---
sequenceDiagram

    % People & Systems
    actor Dev as Developer
    participant CU as Custom Upstream
    participant S0 as Canary Site
    participant S1 as Site 1
    participant S2 as Site 2
    participant S3 as Site 3

    % Actions

    rect rgb(184,192,64)
        Note over Dev,CU: Development Phase
        Dev->>Dev: Code new features in local
        Dev->>CU: Push changes to feature branch
    end

    rect rgb(64,120,192)
        Note over CU,S0: Testing Phase
        CU->>+S0: Deploy for testing
        S0->>-CU: Testing passed
    end
    rect rgb(192,72,64)
        Note over CU,S3: Deployment Phase
        CU->>CU: Merge feature in to master branch
        par Deploying to Site 1
            CU->>S1: Site 1 pulls in upstream updates
            Note over S1: Site 1 goes through its deployment process
        and Deploying to Site 2
            CU->>S2: Site 2 pulls in upstream updates
            Note over S2: Site 2 goes through its deployment process
        and Deploying to Site 3
            CU->>S2: Site 3 pulls in upstream updates
            Note over S3: Site 3 goes through its deployment process
        end
    end

```

```mermaid
---
title: Updating WordPress core
---
sequenceDiagram

    % People & Systems
    participant PU as Pantheon Upstream
    participant CU as Custom Upstream
    participant S0 as Canary Site
    participant S1 as Site 1
    participant S2 as Site 2
    participant S3 as Site 3

    % Actions

    rect rgb(184,192,64)
        Note over PU,CU: General Development Phase
        PU->>PU: New verison of WordPress Core
        PU->>CU: Merge latest updates into Custom Upstream
    end

    rect rgb(64,120,192)
        Note over CU,S0: Testing Phase
        CU->>+S0: Deploy for testing
        S0->>-CU: Testing passed
    end

    rect rgb(192,72,64)
        Note over CU,S3: Deployment Phase
        par Deploying to Site 1
            CU->>S1: Site 1 pulls in upstream updates
            Note over S1: Site 1 goes through its deployment process
        and Deploying to Site 2
            CU->>S2: Site 2 pulls in upstream updates
            Note over S2: Site 2 goes through its deployment process
        and Deploying to Site 3
            CU->>S3: Site 3 pulls in upstream updates
            Note over S3: Site 3 goes through its deployment process
        end
    end

```

```mermaid
---
title: Creating a feature for a single site 
---
sequenceDiagram

    % People & Systems
    actor Dev as Developer
    participant S1MD as Site 1 Multidev
    participant S1DEV as Site 1 Dev
    participant S1TEST as Site 1 Test
    participant S1LIVE as Site 1 Live

    % Actions

    rect rgb(184,192,64)
        Note over Dev,S1MD: Development Phase
        Dev->>Dev: Code new features in local copy of Site 1
        Dev->>S1MD: Push changes to a multidev for testing
    end

    rect rgb(64,120,192)
        Note over S1MD,S1DEV: Testing Phase
        S1MD->>S1MD: Testing in multidev
        S1MD->>S1DEV: Testing passed, merge to Dev
    end

    rect rgb(192,72,64)
        Note over S1DEV,S1LIVE: Deployment Phase
        S1DEV->>S1TEST: Deploy to test, final UAT
        S1TEST->>S1LIVE: Deploy to live
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

### Dependencies in a Monorepo setup

```mermaid
---
title: Dependency chart
---
graph
    UpstreamPantheon["Pantheon Upstream"]:::repo

    subgraph github [GitHub]
        UpstreamCustom["Custom Upstream\nWith parent theme, all child themes, all plugins"]:::repo
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

### Processes for deploying code changes in a monorepo setup

```mermaid
---
title: Release a feature for all sites
---
sequenceDiagram

    % People & Systems
    actor Dev as Developer
    participant CU as Custom Upstream
    participant S0 as Canary Site
    participant S1 as Site 1
    participant S2 as Site 2
    participant S3 as Site 3

    % Actions

    rect rgb(184,192,64)
        Note over Dev,CU: Development Phase
        Dev->>Dev: Code new features in local
        Dev->>CU: Dev pushes to a feature branch in the upstream
    end

    rect rgb(64,120,192)
        Note over CU,S0: Testing Phase
        CU->>+S0: Deploy for testing
        S0->>-CU: Testing passed
    end

    rect rgb(192,72,64)
        Note over CU,S3: Deployment Phase
        par CU to S1
            CU->>S1: Deploy to Site 1
            Note over S1: Site 1 goes through its deployment process
        and CU to S2
            CU->>S2: Deploy to Site 2
            Note over S2: Site 2 goes through its deployment process
        and CU to S3
            CU->>S3: Deploy to Site 3
            Note over S3: Site 3 goes through its deployment process
        end
    end
```

```mermaid
---
title: WordPress Core Update
---
sequenceDiagram

    % People & Systems
    participant PU as Pantheon Upstream
    participant CU as Custom Upstream
    participant S0 as Canary Site
    participant S1 as Site 1
    participant S2 as Site 2
    participant S3 as Site 3

    % Actions

    rect rgb(184,192,64)
        Note over PU,CU: Updates Phase
        PU->>PU: New version of WordPress Core released
        PU->>CU: Merge updates into Custom Upstream
    end

    rect rgb(64,120,192)
        Note over CU,S0: Testing Phase
        CU->>+S0: Deploy for testing
        S0->>-CU: Testing passed
    end

    rect rgb(192,72,64)
        Note over CU,S3: Deployment Phase
        par CU to S1
            CU->>S1: Deploy to Site 1
            Note over S1: Site 1 goes through its deployment process
        and CU to S2
            CU->>S2: Deploy to Site 2
            Note over S2: Site 2 goes through its deployment process
        and CU to S3
            CU->>S3: Deploy to Site 3
            Note over S3: Site 3 goes through its deployment process
        end
    end
```

```mermaid
---
title: Release a feature for a single site
---
sequenceDiagram

    % People & Systems
    actor Dev as Developer
    participant CU as Custom Upstream
    participant S0 as Canary Site
    participant S1 as Site 1
    participant S2 as Site 2
    participant S3 as Site 3

    % Actions
    rect rgb(184,192,64)
        Note over Dev,CU: Development Phase
        Dev->>Dev: Code new feature in local
        Dev->>CU: Add the feature into the custom upstream
    end

    rect rgb(192,72,64)
        Note over CU,S3: Deployment Phase
        par CU to S1
            CU->>S1: Deploy to Site 1
            Note over S1: Site 1 goes through its deployment process. Site 1 uses the feature.
        and CU to S2
            CU->>S2: Deploy to Site 2
            Note over S2: Site 2 goes through its deployment process. Site 2 ignores the feature.
        and CU to S3
            CU->>S3: Deploy to Site 3
            Note over S3: Site 3 goes through its deployment process. Site 3 ignores the feature.
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
