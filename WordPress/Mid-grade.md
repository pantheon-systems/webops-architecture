# Mid-level complexity workflows

## Custom Upstream

In this setup, the company has

- A custom upstream that's based on the Pantheon default WordPress upstream
- The custom upstream has their standard custom theme and suite of plugins that all sites use
- The canary site uses the custom upstream "as-is". It's used for testing changes to the upstream before releasing it to the rest of the sites.
- Each site uses the theme and plugins from the upstream. They can also include a child theme and other plugins that they specifically need.

Useful for

- Teams with a few developers
- The team maintains several websites that all share a common codebase

### Dependency chart showing how the systems connect to each other

```mermaid
---
title: Dependency chart
---
graph LR
    UpstreamPantheon["Pantheon Upstream"]:::repo

    subgraph github [GitHub]
        UpstreamCustom["Custom Upstream\nWith custom theme, plugins"]:::repo
    end

    subgraph pantheon [Pantheon]
        subgraph site0 [Canary Site]
            direction LR
            EnvDev0(Dev Environment):::env
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
    end

    %% Links
    
    UpstreamPantheon --> UpstreamCustom

    UpstreamCustom --> EnvDev0
    UpstreamCustom --> EnvDev1
    UpstreamCustom --> EnvDev2

    EnvDev1 --> EnvTest1
    EnvTest1 --> EnvLive1

    EnvDev2 --> EnvTest2
    EnvTest2 --> EnvLive2

    %% Styles
    style github fill:#4078c0,color:#FFF
    style pantheon fill:#181D21,color:#FFF
    style site0 fill:#78868E,color:#FFF
    style site1 fill:#78868E,color:#FFF
    style site2 fill:#78868E,color:#FFF
    classDef env fill:#EFD01B
    classDef repo fill:#fafafa

```

### Activity Diagram showing how changes move through the systems

```mermaid
---
title: Code deployment activity diagram

---
%%{init:{mirrorActors: false}}%%
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
        Note over Dev,CU: Development Phase
        PU->>CU: WP Core update
        Dev->>Dev: Code new features in local
        Dev->>CU: Dev creates a feature for all sites
        Dev->>S2: Dev creates a feature just for Site 2
    end

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

```

### User Journey showing who's involved at which stages

```mermaid
---
title: Deployment Journey
---
journey
    section Develop new features
        Code new feature in local environment: 1: Developer
        Create PR into Custom Upstream: 2: Developer, Custom Upstream, GitHub
        Update WP core to latest version: 1: Pantheon Upstream, Custom Upstream
        Code new feature just for Site 2: 2: Developer, Pantheon

    section Test features
        Merge PR into Custom Upstream: 3: Developer, Custom Upstream, GitHub
        Deploy changes to Canary Site: 4: Developer, GitHub, Pantheon
        Test Canary site: 4: Tester, Pantheon

    section Deploy features to live site
        Deploy changes to Sites 1, 2: 4:  Developer, GitHub, Pantheon
        Merge changes into Test on Sites 1, 2: 5: Tester, Pantheon
        Merge changes into Live on Sites 1, 2: 5: Tester, Pantheon
```