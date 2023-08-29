# Foundational codebase with compiled assets

Ideal for teams that

- have a lot of team members
- run a number of related, but not identical websites
- require high levels of governance over their sites
- are doing advanced or cutting edge technologies
- do a lot of testing of their codebase

Major elements of this setup

- Foundational codebase including core modules, themes, and configuration
- Can be used for many, many sites (scaling to the hundreds)
- The foundational codebase supports a full testing suite and a frontend build process
- The site repos can add additional modules, themes, and configuration on top of the foundation

```mermaid
---
title: Dependency chart
---
graph
    subgraph github [GitHub]
        subgraph cu [Custom Upstream]
            ContribModules["Contrib modules"]:::repo
            CustomModules["Custom modules"]:::repo
            Theme["Custom Theme"]:::repo
            Config["Configuration"]:::repo
        end
    end

    subgraph ci ["CI System"]
        Workflow1["Generate Build Artifact
        Composer, NPM, Gulp, etc."]
        Workflow2["Run Test Scripts
        PHPCS, Unit, Behat, etc."]
        Workflow3["Deploy to Pantheon
        Uses `*terminus build:env:deploy*`"]
    end

    subgraph pantheon [Pantheon]
        subgraph site [Site]
            EnvDev(Dev Environment):::env
            EnvTest(Test Environment):::env
            EnvLive(Live Environment):::env
        end
    end

    %% Links

        
    cu --> Workflow1 --> Workflow2 --> Workflow3 --> EnvDev

    EnvDev -- Integrated Composer --> EnvTest -- Integrated Composer --> EnvLive

    %% Styles
    style cu fill:#00A0D2,color:#FFF
    style ci fill:#049B4A,color:#FFF
    style github fill:#4078c0,color:#FFF
    style pantheon fill:#181D21,color:#FFF
    style site fill:#78868E,color:#FFF
    classDef env fill:#EFD01B,color:#000
    classDef repo fill:#fafafa,color#000

```

```mermaid
---
title: Sequence for maintaining site code
---
%%{init:{mirrorActors: false}}%%
sequenceDiagram

    % People & Systems
    actor Dev as Developer
    participant CU as Custom Upstream
    participant CI as CI System
    participant EnvPR as PR Multidev
    participant EnvDev as Dev Environment
    participant EnvTest as Test Environment
    participant EnvLive as Live Environment

    % Actions

    rect rgb(64,120,192)
        Note over Dev,EnvLive: Changes to site's codebase
        activate Dev
        Dev->>Dev: Update foundational content
        Dev->>CU: Make PR
        deactivate Dev

        activate CU
        CU->>CI: Trigger off of PR creation
        deactivate CU

        activate CI
        CI->>CI: Run composer install
        CI->>CI: Run frontend build
        CI->>CI: Commit build artifact
        CI->>EnvPR: Push build artifact to PR Multidev
        CI-->>EnvPR: Behat testing
        deactivate CI

        activate EnvPR
        EnvPR->>EnvPR: User acceptance testing
        deactivate EnvPR

        activate CU
        CU->>CU:Merge PR into master branch
        CU->>CI: Trigger off of merge to master
        deactivate CU

        activate CI
        CI->>CI: Run composer install
        CI->>CI: Run PHPCS, unit tests
        CI->>CI: Run frontend build
        CI->>CI: Commit build artifact
        CI->>EnvDev: Push build artifact to Dev
        deactivate CI

        EnvDev->>EnvTest: Deploy to Test
        activate EnvTest
        EnvTest->>EnvTest: Smoke Testing
        EnvTest->>EnvLive: Deploy to Live
        deactivate EnvTest
    end

```
