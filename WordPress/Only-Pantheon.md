# Simple workflows using only the Pantheon framework

## Ongoing changes in Dev environment only

Best for a solo developer working on a website.

The developer edits files on the dev environment using SFTP. They don't have to worry about running a local environment. However, the flow doesn't easily support multiple developers as they will overwrite each others' changes.

Autopilot will update all plugins and the upstream (WordPress Core) on a regular basis. It can either leave the updates in Dev for the developer to deploy or automatically deploy them to Live.

```mermaid
graph TD
    User["fab:fa-user Solo Developer"] == SFTP ==> EnvDev("fas:fa-bolt Dev Environment")
    Upstream["Pantheon Default Upstream"] -- WordPress core updates --> AutoPilot["Autopilot"]
    AutoPilot -. Weekly Update .-> EnvDev
    EnvDev == Commit changes ==> Feat([Ongoing Changes])
    Feat == Deploy to test ==> EnvTest("fas:fa-bolt Test Environment")
    EnvTest == Deploy to live ==> EnvLive("fas:fa-bolt Live Environment")
    classDef env fill:#EFD01B,stroke:#78868E
    class EnvDev,EnvTest,EnvLive env
    classDef user fill:#78868E,color:#FFF
    class User user

```

## Feature development in a multi-dev

Best for a solo developer, or a very small team.

The developer makes ongoing small changes directly on the dev environment using SFTP. For any significant features, they spin up a multidev where that work is isolated.

Autopilot will update all plugins and the upstream (WordPress Core) on a regular basis. It can either leave the updates in Dev for the developer to deploy or automatically deploy them to Live.

```mermaid
graph TD
    User["fab:fa-user Solo Developer"] == SFTP\n Ongoing Changes ==> EnvDev("fas:fa-bolt Dev Environment")
    User["fab:fa-user Solo Developer"] == SFTP\n Feature Development ==> EnvMulti("fas:fa-bolt Multi-dev Environment")
    EnvMulti == Commit changes ==> FeatA([New Feature])
    FeatA == Merge changes ==> EnvDev
    Upstream["Pantheon Default Upstream"] -- WordPress core updates --> AutoPilot["Autopilot"]
    AutoPilot -. Weekly Update .-> EnvDev
    EnvDev == Deploy to test ==> EnvTest("fas:fa-bolt Test Environment")
    EnvTest == Deploy to live ==> EnvLive("fas:fa-bolt Live Environment")
    classDef env fill:#EFD01B,stroke:#78868E
    class EnvMulti,EnvDev,EnvTest,EnvLive env
    classDef user fill:#78868E,color:#FFF
    class User user

```

## Multiple developers isolated in multidevs

Best for small teams who don't develop large amounts of code.

Each developer has their own multidev where they do their work in isolation. After they're ready to deploy their work, they merge it into the Dev environment and then through Test & Live.

If the two developers often work on the same plugins or theme files, then they may run into a number of merge conflicts. Those conflicts can be very hard to resolve if they don't have familiarity with Git and locally cloned copies of the site.

Autopilot will update plugins and the upstream (WordPress Core) on a regular basis. Those updates go into a dedicated multidev. The dev team deploys those updates to dev/test/live after they've reviewed the changes.

```mermaid
graph TD
    UserA["fab:fa-user 1st Developer"] == SFTP ==> EnvMultiA("fas:fa-bolt Multidev Environment")
    UserB["fab:fa-user 2nd Developer"] == SFTP ==> EnvMultiB("fas:fa-bolt Multidev Environment")
    EnvMultiA == Merge when ready ==> EnvDev("fas:fa-bolt Dev Environment")
    EnvMultiB == Merge when ready ==> EnvDev
    Upstream["Pantheon Default Upstream"] -- Dashboard updates --> AutoPilot["Autopilot"]
    AutoPilot -. Weekly Update .-> EnvMultiC("fas:fa-bolt Dedicated Autopilot Multidev")
    EnvMultiC -- Review &\n Manually Merge --> EnvDev
    EnvDev == Deploy to test ==> EnvTest("fas:fa-bolt Test Environment")
    EnvTest == Deploy to live ==> EnvLive("fas:fa-bolt Live Environment")
    classDef env fill:#EFD01B,stroke:#78868E
    class EnvMultiA,EnvMultiB,EnvMultiC,EnvDev,EnvTest,EnvLive env
    classDef user fill:#78868E,color:#FFF
    class UserA,UserB user

```
