# Transformations CI/CD Template

### Why
Transformations in CDF are used to process data from RAW to other CDF resource types ("clean"), for example Assets or Time Series.

### How
This template uses Github Workflows to run the `jetfire-cli` to deploy transformations to CDF on merges to `master`. If you want to use it with multiple CDF projects, e.g. `customer-dev` and `customer-prod`, you can clone the `deploy-push-master.yml` file and modify it for merges to a specific branch of your choice.

### Repository Layout
In the top-level folder `transformations` you may add each transformation job as a new directory, for example:
```
.
├── .github
│   └── workflows
│       ├── deploy-push-master.yml
│       └── check-pr.yml
├── README.md
└── transformations
    ├── my_transformation_001
    │   ├── manifest.yml
    │   └── transformation.sql
    └── my_transformation_002
        ├── manifest.yml
        └── transformation.sql
```
However, you can pretty much change this layout however you see fit - as long as you obey the following rule: Keep one transformation (manifest+sql) per folder.

Example layout:
```
.
└── transformations
    ├── assets
    │   └── my_transformation_001
    │       ├── manifest.yml
    │       └── transformation.sql
    ├── time_series
    │   └── my_transformation_002
    │       ├── manifest.yml
    │       └── transformation.sql
    ├── events
    │   └── my_transformation_003
    │       ├── manifest.yml
    │       └── transformation.sql
    ⋮
    └── raw
        └── my_transformation_004
            ├── manifest.yml
            └── transformation.sql
```

### Requirements
##### Jetfire API-key
In order to connect to CDF, we need the API-key for Jetfire. In this template, it will be automatically read by the workflow, by reading it from your GitHub secrets. Thus, _surprise surprise_, you need to store the API-key in GitHub secrets in your own repo. However, there is one catch! To distinguish between the API-key meant for e.g. testing- and production environments, we control this by appending the branch name responsible for deployment to the end of the secret name like this: `JETFIRE_API_KEY_{BRANCH}`.

Let's check out an example. On merges to 'master', you want to deploy to `customer-dev`, so you use the API-key for this project and store it as a GitHub secret with the name:
`JETFIRE_API_KEY_{BRANCH} -> JETFIRE_API_KEY_MASTER`

Similarly, if you have a `customer-prod` project, and you have created a build-file that only runs on your branch `prod`, you would need to store the API-key to this project under the GitHub secret: `JETFIRE_API_KEY_PROD`. You can of course repeat this for as many projects as you want!

##### Capabilities
The API-key needs the following:
- `groups:list`: To verify that it is a member of the `transformations` or `jetfire` group.

### Manifest

#### Schedule
Gotcha: Since CRON expressions typically use a lot of asterisks, (this fellow: `*`), you have to wrap your schedule in single quotes (`'`) if _it starts with one (asterisk)_!
```yaml
schedule: */2 * * * *  # Not valid
schedule: 0/2 * * * *  # Valid, but non-standard CRON
schedule: '*/2 * * * *'  # Valid
```
