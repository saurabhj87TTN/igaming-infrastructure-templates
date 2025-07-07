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
