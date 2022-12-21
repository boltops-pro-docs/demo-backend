<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/demo-backend/blob/master/docs/route53.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

## Route53 CLI Commands

development route53 record:

    export AWS_PROFILE=dev
    AWS_REGION=$(aws configure get region)
    VPC=vpc-111 # grab the VPC value from the CloudFormation Console Outputs
    aws route53 create-hosted-zone --name private --caller-reference $(date +%Y%m%d%H%M%S) --vpc VPCRegion=$AWS_REGION,VPCId=$VPC --hosted-zone-config PrivateZone=true

production route53 record:

    export AWS_PROFILE=prd
    AWS_REGION=$(aws configure get region)
    VPC=vpc-111 # grab the VPC value from the CloudFormation Console Outputs
    aws route53 create-hosted-zone --name private --caller-reference $(date +%Y%m%d%H%M%S) --vpc VPCRegion=$AWS_REGION,VPCId=$VPC --hosted-zone-config PrivateZone=true

