# Cumulus DAAC

## Overview

This repository contains the configuration and deployment scripts to
deploy Cumulus Core for a DAAC. It is a modified version of the
[Cumulus Template
Deploy](https://github.com/nasa/cumulus-template-deploy)
project. Specifically, all parts of the deployment have been
Terraformed and the configuration minimized by using outputs from
other modules and lookups using Terraform AWS provider data sources.

See the [Cumulus
Documentation](https://nasa.github.io/cumulus/docs/deployment/deployment-readme)
for detailed information about configuring, deploying, and running
Cumulus.

## Prerequisites

* [Docker](https://www.docker.com/get-started)
* One or more NGAP accounts (sandbox, SIT, ...)
* AWS credentials for those account(s)

## Development Setup

You can run tests inside of a Docker container:

        $ make image
        $ make container-shell

1. To run linter (flake8) & unit tests (pytest) once:

        $ make test

2. To run linter & tests when source files change:

        $ make test-watch

## Organization

The repository is organized into three Terraform modules:

* `daac`: Creates DAAC-specific resources necessary for running Cumulus
* `cumulus`: Creates all runtime Cumulus resources that can then be used
  to run ingest workflows.
* `workflows`: Creates a Cumulus workflow with a sample Python lambda.

To customize the deployment for your DAAC, you will need to update
variables and settings in a few of the modules. Specifically:

### daac module

To change which version of the [Cumulus Message
Adapter](https://github.com/nasa/cumulus-message-adapter) is used to
create the Lambda layer used by all Step Function Tasks, modify the
corresponding variable in the `daac/terraform.tfvars` file.

### cumulus module

This module contains the bulk of the DAAC-specific settings. There are
three specific things you should customize:

* `cumulus/terraform.tfvars`: Variables which are likely the same in all
  environments (SIT, UAT, PROD) _and_ which are not 'secrets'.

* `cumulus/variables/*.tfvars`: Each file contains variables specific to
  the corresponding 'maturity' or environment to which you are
  deploying. For example, in `dev.tfvars` you will likely use a
  pre-production `urs_url`, while in the `prod.tfvars` file you will
  specify the production url.

* `cumulus/secrets/*.tfvars`: Like the variables above, these files
  contains *secrets* which are specific to the 'maturity' or environment
  to which you are deploying. Create one file for each environment and
  populate it with secrets. See the example file in this directory for
  a starting point. For example, your `dev` `urs_client_password` is
  likely (hopefully!) different than your `prod` password.

*Important Note*: The secrets files will *not* (and *should not*) be
committed to git. The `.gitignore` file will ignore them by default.

### workflows module

DAAC-specific workflows, lambdas, and configuration will be deployed
by this module. Most workflow development work will be done here.

## Deploying Cumulus

See [CIRRUS-core README](https://github.com/asfadmin/CIRRUS-core/blob/master/README.md).

## Developing Cumulus Workflows

There is a sample Workflow Terraform module in the `workflows`
directory. It deploys a `NOP` (No Operation) lambda and workflow. You
can use this as a base for deploying your own workflows. It includes a
Python lambda with unit tests. You can run the tests as shown above.

## Deploying dashboard to S3 bucket

There is a `dashboard` make target which will build and deploy a version of a
Cumulus dashboard to a bucket named `$DEPLOY_NAME-cumulus-$MATURITY-dashboard`
assuming you such a bucket.

You need to pass in:

        CUMULUS_API_ROOT="your api root"
        CUMULUS_DASHBOARD_VERSION="version-of-dashboar"
        DEPLOY_NAME=your deploy name
        MATURITY=dev
        SERVED_BY_CUMULUS_API=true (optional defaults to true)

Example

        $ CUMULUS_API_ROOT="https://xxx.execute-api.us-west-2.amazonaws.com:8000/dev" \
          CUMULUS_DASHBOARD_VERSION="v1.8.0" \
          DEPLOY_NAME=kb \
          MATURITY=dev \
          SERVED_BY_CUMULUS_API=false \
          make dashboard
