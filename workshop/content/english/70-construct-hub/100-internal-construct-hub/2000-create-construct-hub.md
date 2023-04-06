+++
title = "Create Construct Hub"
weight = 200
+++

## Create Construct Hub Infrastructure

{{% notice warning %}} Please note that the Construct Hub web interface is delivered through a publicly accessible CloudFront distribution. Restricting access to specific users and groups is beyond the scope of this workshop. However, you may consider the following best practices to implement it:

- Use signed URLs or signed cookies to grant temporary access to the content in your CloudFront distribution.
- Use AWS Web Application Firewall (WAF) to protect your CloudFront distribution from common web attacks and unwanted traffic. You can create custom rules to block or allow access based on specific criteria, such as IP addresses, user agents, or geographic locations.
- Use Lambda@Edge to add custom logic to your CloudFront distribution. You can use Lambda@Edge to perform authorization checks and allow or deny access to your content based on specific criteria.
- For access inside of an Intranet or private networks, disable your CloudFront distribution and provide access to the origin S3 bucket through an internal Application Load Balancer using interface endpoints on AWS PrivateLink for Amazon S3.
  {{% /notice %}}

{{% notice warning %}}
Before you begin, make sure you have gone through the steps in the [Prerequisites](/15-prerequisites.html) section.

You must also have Docker running and Yarn installed in your dev environment to complete this walkthrough.
{{% /notice %}}

### Create Construct Hub Stack

As an Internal Construct Hub Administrator, the first step is to create an instance of Construct Hub in an AWS Account. Before we can use the Construct Hub library in our stack, we need to install the npm module:

{{<highlight bash>}}
npm install construct-hub
{{</highlight>}}

By default, Construct Hub has a single package source configured, which is the public npmjs.com registry. However, it also supports CodeArtifact repositories and custom package source implementations. For our purposes, we will create a CodeArtifact domain and repository to add as a package source to our internal Construct Hub.

Create a new file under `lib/internal-construct-hub-stack.ts` with the following code:

{{<highlight ts>}}
import * as cdk from 'aws-cdk-lib';
import * as codeartifact from 'aws-cdk-lib/aws-codeartifact';
import { ConstructHub } from 'construct-hub';
import * as sources from 'construct-hub/lib/package-sources';
import { Construct } from 'constructs';

export class InternalConstructHubStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create a CodeArtifact domain and repo for user construct packages
    const domain = new codeartifact.CfnDomain(this, 'CodeArtifactDomain', {
      domainName: 'cdkworkshop-domain',
    });

    const repo = new codeartifact.CfnRepository(this, 'CodeArtifactRepository', {
      domainName: domain.domainName,
      repositoryName: 'cdkworkshop-repository',
    });

    repo.addDependency(domain);

    // Create internal instance of ConstructHub, register the new CodeArtifact repo
    new ConstructHub(this, 'ConstructHub', {
      packageSources: [
        new sources.CodeArtifact({ repository: repo })
      ],
    });
  }
}
{{</highlight>}}

## Bootstrapping an environment
The first time you deploy an AWS CDK app into an environment (account/region),
you can install a "bootstrap stack". This stack includes resources that
are used in the toolkit's operation. For example, the stack includes an S3
bucket that is used to store templates and assets during the deployment process.

You can use the `cdk bootstrap` command to install the bootstrap stack into an
environment:

```
cdk bootstrap
```

If your environment is already bootstrapped, cdk will tell you:

{{<highlight bash>}}
Environment aws://[account-id]/[region] bootstrapped (no changes).
{{</highlight>}}


### Deploy

Use `npx cdk deploy` to deploy the CDK app:

```
npx cdk deploy
```

{{% notice info %}} Deploying Construct Hub stack for the first time may take up to 10-12 minutes. {{% /notice %}}