<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/demo-backend/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# Demo Backend App

[![Watch the video](https://img.boltops.com/boltopspro/video-preview/multiple/demo-frontend.png)](https://youtu.be/-GZGBY8PbnQ)

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

This backend app that is useful for demonstration purposes.  It simply provides an endpoint that returns the number 42. It is a Sinatra Ruby app.

## Install

To install the dependencies.

    git clone git@github.com:boltopspro/demo-backend.git
    cd demo-backend
    bundle

## Start Server Locally

Start app with `bin/web`. Example:

    $ bin/web
    == Sinatra (v2.0.5) has taken the stage on 8080 for development with backup from Puma
    Puma starting in single mode...
    * Version 3.12.0 (ruby 2.5.3-p105), codename: Llamas in Pajamas
    * Min threads: 0, max threads: 16
    * Environment: development
    * Listening on tcp://0.0.0.0:8080
    Use Ctrl-C to stop

The backend app runs on port 8080.

Test with curl:

    $ curl localhost:8080
    42

## Create ECR Repos

Let's get ready to deploy to ECS.  We'll use AWS_PROFILE to switch between accounts. More info: [Settings Setup](https://github.com/boltopspro-docs/reference-architecture/blob/master/docs/settings-setup.md).

Create the ECR repos on both AWS acccounts.

    AWS_PROFILE=dev aws ecr create-repository --repository-name demo/backend
    AWS_PROFILE=prd aws ecr create-repository --repository-name demo/backend

If you need to list the repo again:

    AWS_PROFILE=dev aws ecr describe-repositories
    AWS_PROFILE=prd aws ecr describe-repositories

If you are using a Single AWS Account, use the commands: [One Account: Create ECR Repos](docs/one-account.md#create-ecr-repos).

## ECS Deployment

Let's deploy the demo app to an ECS cluster. We'll use the [ufo](http://ufoships.com/) tool for deployment.  The project has already been set up with starter `.ufo` files for you. We'll just need to adjust some of the `.ufo` settings.

### Adjust ECR Repos

Adjust the `.ufo/settings.yml` file, which looks like this:

```yaml
development:
  image: 112233445566.dkr.ecr.us-west-2.amazonaws.com/demo/backend
  aws_profile: dev

production:
  image: 778899001122.dkr.ecr.us-west-2.amazonaws.com/demo/backend
  aws_profile: prd
```

Adjust the `.ufo/settings.yml` file to use the ECR repos that were created in the previous step.

If you are using a Single AWS Account, follow the instructions in: [One Account: Adjust ECR Repos](docs/one-account.md#adjust-ecr-repos).

## Adjust Network Settings

Next, adjust the network settings that the ECS service will use.

Since the backend app is meant to be an internal application, everything should run on `PrivateApp` subnets.  Both the ELB and the ECS tasks will be placed on `PrivateApp` subnets.  Adjust the `.ufo` network related files:

* [.ufo/settings/network/development.yml](.ufo/settings/network/development.yml)
* [.ufo/settings/network/production.yml](.ufo/settings/network/production.yml)

Here's what the [network/development.yml](.ufo/settings/network/development.yml) settings file looks like:

```yaml
vpc: vpc-111
ecs_subnets: # required: at least 2 subnets
  - subnet-111,subnet-222,subnet-333
elb_subnets: # defaults to same subnets as ecs_subnets when not set
  - subnet-111,subnet-222,subnet-333
```

We can grab the values from the CloudFormation Console Outputs tab of the VPC. Here's an example of the development VPC.

![](https://img.boltops.com/boltopspro/demo-apps/backend/dev-vpc-outputs.png)

Repeat the process for the [network/production.yml](.ufo/settings/network/production.yml) file.

## Internal Route53 Record

Creating an internal route53 record is optional but is recommended so you have a human-friendly name to refer to the internal backend server like http://demo-backend-web.dev.private/  Refer to the AWS [Creating a Private Hosted Zone](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zone-private-creating.html) docs to create a private hosted zone named `private`.  Here's a screenshot of the Route53 console interface for the development environment:

![](https://img.boltops.com/boltopspro/demo-apps/backend/route53-dev-private.png)

Create the Route53 Private Hosted Zone for records the production environment also.

Adjust these `.ufo` CloudFormation template related files so that ufo will create and manage the route53 DNS endpoints.

* [.ufo/settings/cfn/development.yml](.ufo/settings/cfn/development.yml)
* [.ufo/settings/cfn/production.yml](.ufo/settings/cfn/production.yml)

There is already an commented out dns entry at the bottom. You just have to uncomment the setting. Here's the [cfn/development.yml](.ufo/settings/cfn/development.yml) setting:

```yaml
dns:
  name: "{stack_name}.dev.private."
  hosted_zone_name: dev.private. # dont forget the trailing period
  TTL: '60' # ttl has special upcase casing
```

This setting tells ufo to create the `demo-backend-web.dev.private` route53 record and point it to the internal ELB.  Repeat the process for the production account with the [cfn/production.yml](.ufo/settings/cfn/production.yml) file.

If you are using a Single AWS Account, follow the instructions in: [One Account: Internal Route53 Record](docs/one-account.md#internal-route53-record).

## Deploy with ufo ship

We're ready to deploy the application with [ufo ship](https://ufoships.com/reference/ufo-ship/). For convenience, we'll provide example commands for both development and production environments.

    UFO_ENV=development ufo ship demo-backend-web --no-wait
    UFO_ENV=production  ufo ship demo-backend-web --no-wait

Check the CloudFormation and ECS console for the deployment to complete. It usually take about 5 to 6 minutes. After deployment completes you can grab the DNS endpoint from the ufo managed CloudFormation stack Outputs.  Here's an screenshot example of development:

![](https://img.boltops.com/boltopspro/demo-apps/backend/dev-ufo-outputs.png)

You can also use the `ufo ps` to get the internal ELB DNS endpoint.  Examples:

    $ UFO_ENV=development ufo ps demo-backend-web
    Elb: internal-demo-ba-Elb-AAABBBEXAMPLE-1122334455.us-west-2.elb.amazonaws.com
    Route53: demo-backend-web.dev.private
    $ UFO_ENV=production  ufo ps demo-backend-web
    Elb: internal-demo-ba-Elb-XXXYYYEXAMPLE-1122334455.us-west-2.elb.amazonaws.com
    Route53: demo-backend-web.prd.private

## Test Deployed App

Since we created an internal ELB, we must be inside the VPC network to reach it. The instances that were created by the [ecs-spot](https://github.com/boltopspro-docs/ecs-spot) or [ecs-asg](https://github.com/boltopspro-docs/ecs-asg) blueprints are inside the VPC network.

Use session-manager hop onto the instance.  Here's an example checking the development environment.

    $ AWS_PROFILE=dev aws ssm start-session --target i-1122334455
    # curl http://internal-demo-ba-Elb-AAABBBEXAMPLE-1122334455.us-west-2.elb.amazonaws.com
    42
    # curl http://demo-backend-web.dev.private
    42
    #

Repeat the process for the production account.

## Go Back to Reference Architecture

That's it. Go back to the main [boltopspro/reference-architecture](https://github.com/boltopspro-docs/reference-architecture/blob/master/README.md)
