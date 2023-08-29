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
- If the sites need a frontend build step, that build must be done locally and have the compiled assets committed locally & pushed to Pantheon
- Terminus Build Tools workflow commands are generally incompatible

```mermaid
---
title: Dependency chart
---
graph
    subgraph github [GitHub]
        subgraph cu [Custom Upstream]
            Modules["Contrib modules"]:::repo
            CustomModules["Custom modules"]:::repo
            Themes["Contrib themes"]:::repo
            Config["Configuration"]:::repo
        end
    end

    subgraph local [Local Machine]
        subgraph localcu [Custom Upstream]
            LocalModules["Contrib modules"]:::repo
            LocalCustomModules["Custom modules"]:::repo
            LocalThemes["Contrib themes"]:::repo
            LocalConfig["Configuration"]:::repo
        end
        subgraph localsite1 [Site 1]
            SiteConfig1[Site Config]:::repo
            SiteTheme1[Site Theme]:::repo
        end

        subgraph localsite2 [Site 2]
            SiteConfig2[Site Config]:::repo
            SiteTheme2[Site Theme]:::repo
            SiteModules2[Extra contrib modules]:::repo
        end
    end

    subgraph ci ["CI System"]
        Workflow1["Build
        Composer, NPM, Gulp, etc."]
        Workflow2["Test
        PHPCS, Unit, Behat, etc."]
        Workflow3["Deploy"]
    end

    subgraph pantheon [Pantheon]
        subgraph site1 [Site 1]
            subgraph env1 [Environments]
                EnvIntegration1(Integration Multidev):::env
                EnvDev1(Dev Environment):::env
                EnvTest1(Test Environment):::env
                EnvLive1(Live Environment):::env
            end
            subgraph custom1 [Customizations]
                theme1(Custom Theme)
                config1(Site specific config)
            end
        end

        subgraph site2 [Site 2]
            subgraph env2 [Environments]
                EnvIntegration2(Integration Multidev):::env
                EnvDev2(Dev Environment):::env
                EnvTest2(Test Environment):::env
                EnvLive2(Live Environment):::env
            end
            subgraph custom2 [Customizations]
                theme2(Custom Theme)
                modules2(Contrib Modules)
                config2(Site specific config)
            end
        end

        subgraph site3 [Site 3]
            subgraph env3 [Environments]
                EnvIntegration3(Integration Multidev):::env
                EnvDev3(Dev Environment):::env
                EnvTest3(Test Environment):::env
                EnvLive3(Live Environment):::env
            end
            subgraph custom3 [Customizations]
                config3(Site specific config)
            end
        end
    end

    %% Links
    localcu --> cu
    cu --> Workflow1

    Workflow1 --> Workflow2 --> Workflow3

    Workflow3 --> EnvIntegration1
    Workflow3 --> EnvIntegration2 
    Workflow3 --> EnvIntegration3

    localsite1 -- Local build & commit --> EnvDev1
    localsite2 -- Local build & commit --> EnvDev2

    EnvIntegration1 -- Integrated Composer --> EnvDev1 -- Integrated Composer --> EnvTest1 -- Integrated Composer --> EnvLive1
    
    EnvIntegration2 -- Integrated Composer --> EnvDev2 -- Integrated Composer --> EnvTest2 -- Integrated Composer --> EnvLive2
    
    EnvIntegration3 -- Integrated Composer --> EnvDev3 -- Integrated Composer --> EnvTest3 -- Integrated Composer --> EnvLive3


    %% Styles
    style cu fill:#00A0D2,color:#FFF
    style ci fill:#049B4A,color:#FFF
    style github fill:#4078c0,color:#FFF
    style pantheon fill:#181D21,color:#FFF
    style site1 fill:#78868E,color:#FFF
    style site2 fill:#78868E,color:#FFF
    style site3 fill:#78868E,color:#FFF
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
    participant EnvInt as Integration Multidev
    participant EnvDev as Dev Environment
    participant EnvTest as Test Environment
    participant EnvLive as Live Environment

    % Actions

    rect rgb(64,120,192)
        Note over Dev,EnvLive: Make updates in Foundational codebase
        activate Dev
        Dev->>Dev: Update foundational content
        Dev->>CU: Make PR
        deactivate Dev
        
        activate CU
        Note over CU,CI: Triggered off of merge to master
        CU->>CI: Trigger deploy workflow
        deactivate CU

        activate CI
        CI->>CI: Run composer install
        CI->>CI: Run frontend build
        CI->>CI: Commit foundation's compiled frontend assets
        deactivate CI

        activate CI
        CI->>CI: Run PHPCS, unit, behat tests
        deactivate CI

        CI->>EnvDev: Push compiled codebase to Integration

        EnvInt->>EnvDev: Deploy to Dev
        EnvDev->>EnvTest: Deploy to Test
        activate EnvTest
        EnvTest->>EnvTest: User acceptance Testing
        EnvTest->>EnvLive: Deploy to Live
        deactivate EnvTest
    end

    rect rgb(254,98,29)
        Note over Dev,EnvLive: Make site specific changes
        activate Dev
        Dev->>Dev: Update site specific content
        Dev->>Dev: Run frontend build
        Dev->>Dev: Commit site-specific compiled frontend assets
        Dev->>EnvDev: Push changes to Dev Environment
        deactivate Dev

        EnvDev->>EnvTest: Deploy to Test
        activate EnvTest
        EnvTest->>EnvTest: User acceptance Testing
        EnvTest->>EnvLive: Deploy to Live
        deactivate EnvTest
    end

```
