# Integrated Composer (IC)

At its most simple, it runs `composer install` command when you push code to the Pantheon platform.

IC generates two build artifacts. One includes dev dependencies. One excludes dev dependencies. The dev build gets used for the multidev or dev environment that triggered IC. The production build gets used for test and live environments.

```mermaid
---
title: Code deployment activity diagram
---
sequenceDiagram

    % People & Systems
    actor User as Developer
    participant IC as Integrated Composer
    participant MD as Multidev Environment
    participant Dev as Dev Environment
    participant Test as Test Environment
    participant Live as Live Environment

    % Actions
    alt Working in Multidev
        User ->> MD: Push code updates
        MD -->> IC: Trigger Composer build
        IC ->> IC: Generate dev build
        IC -->> MD: Push build to multidev
    else Deploying to Live
        User ->> Dev: Push code updates
        Dev ->> IC: Trigger Composer build
        IC ->> IC: Generate 2 builds: dev & production
        IC ->> Dev: Push dev build to dev environment
        IC ->> Test: Push production build to test
        IC ->> Live: Push production build to live
    end
```