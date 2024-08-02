# Federated Codebase

A federated codebase refers to one in which there's no controlling upstream that dictates the code used by individual sites. Instead, your organization maintains a collection of plugins, themes, and other features - each of which is maintained seperately. Individual sites can "pick and choose" between the available packages to use the ones that they need.

This model gives a high level of control to individual sites, with the parent organization providing options rather than dictating a particular set of packages.

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

## High level dependency mapping

```mermaid
---
title: Dependency chart
---
flowchart TD
    subgraph contrib [Contributed]
        Contrib0["WP Core"]
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
            Custom5["Shared CI scripts"]:::repo
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

## Process for releasing a new version of a custom theme

```mermaid
---
title: Add new feature to analytics plugin

---
%%{init:{mirrorActors: false}}%%
sequenceDiagram

    % People & Systems
    actor Dev as Developer
    participant Theme as Custom Theme Repo
    participant CI as CI/CD System

    % Actions

    rect rgb(184,192,64)
        Note over Dev,CI: Create new version of theme
        activate Dev
        Dev->>Dev: Code new features in local
        Dev->>Theme: Make PR
        deactivate Dev

        Theme->>CI: Trigger testing workflow
        activate CI
        CI-->Theme: Report success
        deactivate CI

        activate Theme
        Theme-->Theme: Code review
        Theme-->Theme: Release a new version
        Theme->>CI: Trigger release workflow
        deactivate Theme

        activate CI
        CI-->CI: Compile CSS and JS
        CI->>Theme: Provide compiled & packaged code
        deactivate CI

        Note over Theme: Latest theme version is now available to sites
    end

```

## Process for updating plugins on a site

```mermaid
---
title: Add new feature to analytics plugin

---
%%{init:{mirrorActors: false}}%%
sequenceDiagram

    % People & Systems
    participant Contrib as WPPackagist
    participant Plugin as Analytics Plugin Repo
    participant Site as Site Repo
    participant CI as CircleCI
    participant EnvDev as Dev Environment
    participant EnvTest as Test Environment
    participant EnvLive as Live Environment

    % Actions

    rect rgb(64,120,192)
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

## Example composer.json for a custom package

You'll need to include a composer.json file to make a repo available for other sites to install. This is an example of what that would look like for a custom parent theme.

```json
{
    "name": "my-organization/theme",
    "version": "1.0.0",
    "description": "Common parent theme for my-organization",
    "type": "wordpress-theme",
    "require": {
        "composer/installers": "~2.0"
    }
}
```

## Example composer.json for a single site

This is an example composer.json file for a WordPress site. It installs the Bedrock version of WordPress core, some required Pantheon packages, a few plugins from the WordPress plugins repo, and the orgnaization's custom parent theme.

```json
{
    "name": "my-organization/shopping-site",
    "description": "Online store repository",
    "type": "project",
    "repositories": {
        "wpackagist": {
            "type": "composer",
            "url":"https://wpackagist.org",
            "only": [
                "wpackagist-plugin/*",
                "wpackagist-theme/*"
            ]
        },
        "my-org-theme": {
            "type": "github",
            "url": "git@github.com:my-organization/theme"
        }
    },
    "require": {
        "php": ">=8.0",
        "vlucas/phpdotenv": "^5.5",
        "oscarotero/env": "^2.1",
        "pantheon-systems/pantheon-mu-plugin": "*",
        "roots/bedrock-autoloader": "*",
        "roots/bedrock-disallow-indexing": "*",
        "roots/wordpress": "*",
        "roots/wp-config": "*",
        "roots/wp-password-bcrypt": "*",
        "wpackagist-plugin/akismet": "^5.3",
        "wpackagist-plugin/wordpress-seo": "^23.1",
        "wpackagist-plugin/woocommerce": "^9.1",
        "wpackagist-theme/twentytwentytwo": "^1.2",
        "my-organization/theme": "^1.0"
    },
    "config": {
        "allow-plugins": {
            "composer/installers": true
        }
    }
}
```
