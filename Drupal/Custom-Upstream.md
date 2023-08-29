# Custom upstream

Ideal for teams that

- run a number of related, but not identical websites
- require high levels of governance over their sites
- do not use SASS or modern JS frameworks
- do not use automated testing systems

Major elements of this setup

- Foundational codebase including core modules, themes, and configuration
- Can be used for many, many sites (scaling to the hundreds)
- The site repos can add additional modules, themes, and configuration on top of the foundation

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

    subgraph pantheon [Pantheon]
        subgraph site1 [Site 1]
            subgraph env1 [Environments]
                EnvDev1(Dev Environment):::env
                EnvTest1(Test Environment):::env
                EnvLive1(Live Environment):::env
            end
            subgraph custom1 [Customizations]
                theme1(Contrib modules)
                config1(Site specific config)
            end
        end

        subgraph site2 [Site 2]
            subgraph env2 [Environments]
                EnvDev2(Dev Environment):::env
                EnvTest2(Test Environment):::env
                EnvLive2(Live Environment):::env
            end
            subgraph custom2 [Customizations]
                theme2(Custom Modules)
                modules2(Contrib Modules)
                config2(Site specific config)
            end
        end

        subgraph site3 [Site 3]
            subgraph env3 [Environments]
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
    cu --> EnvDev1 -- Integrated Composer --> EnvTest1 -- Integrated Composer --> EnvLive1
    
    cu --> EnvDev2 -- Integrated Composer --> EnvTest2 -- Integrated Composer --> EnvLive2
    
    cu --> EnvDev3 -- Integrated Composer --> EnvTest3 -- Integrated Composer --> EnvLive3


    %% Styles
    style cu fill:#00A0D2,color:#FFF
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
    participant EnvDev as Dev Environment
    participant EnvTest as Test Environment
    participant EnvLive as Live Environment

    % Actions

    rect rgb(64,120,192)
        Note over Dev,EnvLive: Make updates in Custom upstream
        activate Dev
        Dev->>CU: Update upstream content
        deactivate Dev
        
        EnvDev->>CU: Check for updated content
        CU-->>EnvDev: Pull updated content into Dev

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
        Dev->>EnvDev: Push changes to Dev Environment
        deactivate Dev

        EnvDev->>EnvTest: Deploy to Test
        activate EnvTest
        EnvTest->>EnvTest: User acceptance Testing
        EnvTest->>EnvLive: Deploy to Live
        deactivate EnvTest
    end

```
