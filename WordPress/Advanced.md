# Advanced Workflows

## Fully CI controlled

Ideal for teams that

- have a lot of team members
- run a number of related, but not identical websites
- require high levels of governance over their sites
- are doing advanced or cutting edge technologies
- do a lot of testing of their codebase

Major elements of this setup

- Custom plugins and themes are in their own repo
- The plugins & themes store pre-compiled versions of themselves
- The sites use Composer to define which plugins should get pulled in, including the custom plugins
- The site repos may just be the composer.json file and a few other config files
- CircleCI does all the heavy lifting moving code around

```mermaid
---
title: Dependency chart
---
graph LR
    subgraph contrib [Contributed]
        Contrib1["Yoast"]
        Contrib2["WooCommerce"]
        Contrib3["Gravity Forms"]
        Contrib4["WordFence"] 
    end

    subgraph github [GitHub]
        subgraph customCode [Custom Code]
            Custom1["Custom analytics plugin"]:::repo
            Custom2["Custom integration plugin"]:::repo
            Custom3["Custom forms plugin"]:::repo
            Custom4["Custom theme"]:::repo
        end
        SiteCode1["Site 1"]:::repo
        SiteCode2["Site 2"]:::repo
        SiteCode3["Site 3"]:::repo
    end

    subgraph ci [CircleCI]
        Workflow1["Build"]
        Workflow2["Test"]
        Workflow3["Deploy"]
    end

    subgraph pantheon [Pantheon]
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
    EnvDev1 --> EnvTest1 --> EnvLive1
    EnvDev2 --> EnvTest2 --> EnvLive2
    EnvDev3 --> EnvTest3 --> EnvLive3

    Workflow1 --> Workflow2 --> Workflow3

    contrib --> SiteCode1
    customCode --> SiteCode1

    contrib --> SiteCode2
    customCode --> SiteCode2

    contrib --> SiteCode3
    customCode --> SiteCode3

    SiteCode1 --> Workflow1
    SiteCode2 --> Workflow1
    SiteCode3 --> Workflow1

    Workflow3 --> EnvDev1
    Workflow3 --> EnvDev2 
    Workflow3 --> EnvDev3

    %% Styles
    style contrib fill:#00A0D2,color:#FFF
    style ci fill:#049B4A,color:#FFF
    style github fill:#4078c0,color:#FFF
    style customCode fill:#4078c0,color:#FFF
    style pantheon fill:#181D21,color:#FFF
    style site1 fill:#78868E,color:#FFF
    style site2 fill:#78868E,color:#FFF
    style site3 fill:#78868E,color:#FFF
    classDef env fill:#EFD01B
    classDef repo fill:#fafafa

```

### Activity Diagram showing how changes move through the systems

```mermaid
---
title: Add new feature to analytics plugin

---
%%{init:{mirrorActors: false}}%%
sequenceDiagram

    % People & Systems
    participant Contrib as WPPackagist
    actor Dev as Developer
    participant Plugin as Analytics Plugin Repo
    participant Site as Site Repo
    participant CI as CircleCI
    participant EnvDev as Dev Environment
    participant EnvTest as Test Environment
    participant EnvLive as Live Environment

    % Actions

    rect rgb(184,192,64)
        Note over Dev,CI: Create new version of plugin
        activate Dev
        Dev->>Dev: Code new features in local
        Dev->>Plugin: Make PR
        deactivate Dev

        Plugin->>CI: Trigger testing workflow
        activate CI
        CI-->Plugin: Report success
        deactivate CI

        activate Plugin
        Plugin-->Plugin: Code review
        Plugin-->Plugin: Release a new version
        Plugin->>CI: Trigger release workflow
        deactivate Plugin

        activate CI
        CI-->Plugin: Provide compiled & packaged code
        deactivate CI
    end

    rect rgb(64,120,192)
        Note over Plugin,EnvLive: Update plugin on site
        activate Site
        Note over Site: Either scheduled update or manual trigger
        Site->>CI: Trigger deploy workflow
        deactivate Site

        activate CI

            activate CI
            CI->>CI: Run build workflow
                CI->>Plugin: Fetch custom plugin
                activate Plugin
                Plugin-->>CI: CI pulls latest release for plugin
                deactivate Plugin

                CI->>Contrib: Fetch all other plugins
                activate Contrib
                Contrib-->>CI: CI pulls contributed plugins
                deactivate Contrib
            deactivate CI

            activate CI
            CI->>CI: Run test workflow
            deactivate CI

            activate CI
            CI->>CI: Run deploy workflow
            CI->>EnvDev: Push compiled codebase to Dev
            deactivate CI

        deactivate CI

        EnvDev->>EnvTest: Deploy to Test
        activate EnvTest
        EnvTest->>EnvTest: User acceptance Testing
        EnvTest->>EnvLive: Deploy to Live
        deactivate EnvTest
    end

```
