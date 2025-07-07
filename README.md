# 1. Frontend Profile (Static Web Hosting)
Creates:

✅ S3 Bucket → For hosting static site files (HTML/CSS/JS).

✅ CloudFront Distribution → Global CDN for serving the content with caching & HTTPS.

✅ CloudFront Origin Access Identity (OAI) → Secures direct S3 access.

## Interconnection:

CloudFront pulls content from S3 using the OAI. Users always access the site via CloudFront URL.

## Sequence:

Can be created independently but must upload site files after deployment.

# 2. Core Engine / Gaming Layer (ECS Fargate)
Creates:

✅ ECS Cluster → Logical grouping for container services.

✅ IAM Role → Grants ECS permissions to pull images, write logs, etc.

✅ Fargate Task Definition → Container blueprint (CPU/mem/image).

✅ Fargate Service → Runs and manages containers across subnets.

## Interconnection:

ECS Service runs inside your VPC, requires Subnets and Security Groups (must be passed as parameters or managed externally).

## Sequence:

This stack depends on having the VPC, subnets, security groups ready—should be deployed after any core networking stack.

# 3. Datastore Profile (RDS Database)
Creates:

✅ RDS Instance (PostgreSQL) → Managed relational database with encryption.

## Interconnection:

ECS (Core Engine) or Lambda functions will connect to this DB privately via VPC security groups.

## Sequence:

Can be created independently, but the RDS SG rules must allow access from the ECS/Lambda services that need the DB.

# 4. External Systems Profile (PrivateLink + VPC Endpoint)
Creates:

✅ Interface VPC Endpoint → PrivateLink to connect securely to AWS-managed services (S3 here as example).

## Interconnection:

ECS, Lambda, or any service inside the VPC can use PrivateLink to access external AWS services without going over the internet.

Sequence:

Should be created before any workloads that need external API access or private services.

# 5. Integrations Profile (Event-Driven: SQS + Lambda)
Creates:

✅ SQS Queue → Messaging backbone for event-driven systems.

✅ Lambda Function → Consumes messages from the queue.

✅ IAM Role for Lambda → Permissions to interact with SQS.

✅ Event Source Mapping → Links SQS and Lambda.

## Interconnection:

Other systems (ECS, external services) send messages to SQS.

Lambda processes them asynchronously.

## Sequence:

Can be deployed standalone. The producers that send messages to SQS must be configured after this stack is deployed.


----------
# How to run the Steps to install the Infra components:

## Prerequisites:
1. AWS CLI installed and configured (aws configure)
2. AWS CloudFormation permissions (AdministratorAccess or scoped permissions)
3. VPC, Subnet IDs, and Security Group IDs ready (for ECS, RDS, PrivateLink)

## Deployment Sequence & Commands:
### 1. Deploy External Systems Profile (VPC Endpoint)
```bash
  aws cloudformation create-stack \
  --stack-name external-systems-stack \
  --template-body file://external-systems.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters ParameterKey=VpcId,ParameterValue=vpc-xxxxxxx \
               ParameterKey=SubnetIds,ParameterValue='["subnet-xxxxxx"]' \
               ParameterKey=SecurityGroupIds,ParameterValue='["sg-xxxxxx"]'
```

### 2. Deploy Datastore Profile (RDS)

```bash
aws cloudformation create-stack \
  --stack-name datastore-stack \
  --template-body file://datastore.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

### 3. Deploy Integrations Profile (SQS + Lambda)
```bash
aws cloudformation create-stack \
  --stack-name integrations-stack \
  --template-body file://integrations.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

### 4. Deploy Core Engine Profile (ECS Fargate)
```bash
aws cloudformation create-stack \
  --stack-name core-engine-stack \
  --template-body file://core-engine.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters ParameterKey=SubnetIds,ParameterValue='["subnet-xxxxxx"]' \
               ParameterKey=SecurityGroupIds,ParameterValue='["sg-xxxxxx"]'
```

### 5. Deploy Frontend Profile (S3 + CloudFront)
```bash
aws cloudformation create-stack \
  --stack-name frontend-stack \
  --template-body file://frontend.yaml
