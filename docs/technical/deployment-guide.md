# Deployment & Infrastructure Guide

## ☁️ Cloud Infrastructure Architecture

### Infrastructure Overview
Elite Locadora utilizes a cloud-native architecture designed for scalability, reliability, and security. The infrastructure is built on AWS (Amazon Web Services) with multi-region deployment capabilities for high availability and disaster recovery.

### High-Level Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                     Internet Gateway                        │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────┴───────────────────────────────────────┐
│                  Application Load Balancer                  │
│                     (ALB with WAF)                         │
└─────────────┬───────────────────────────┬───────────────────┘
              │                           │
    ┌─────────┴──────────┐      ┌────────┴──────────┐
    │   Web Tier (ECS)   │      │  API Tier (ECS)   │
    │   - React Frontend │      │  - Laravel Backend │
    │   - Nginx Proxy    │      │  - Redis Cache     │
    └─────────┬──────────┘      └────────┬──────────┘
              │                          │
    ┌─────────┴──────────────────────────┴──────────┐
    │              Database Tier                     │
    │         - RDS MySQL (Multi-AZ)                │
    │         - ElastiCache Redis Cluster           │
    │         - S3 for File Storage                 │
    └────────────────────────────────────────────────┘
```

## 🚀 Deployment Strategies

### Container Orchestration with ECS
**Elastic Container Service (ECS) Configuration:**

#### Frontend Service (React Application)
```yaml
# docker-compose.yml for frontend
version: '3.8'
services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    ports:
      - "80:80"
      - "443:443"
    environment:
      - REACT_APP_API_URL=${API_URL}
      - REACT_APP_ENVIRONMENT=${ENVIRONMENT}
      - REACT_APP_STRIPE_KEY=${STRIPE_PUBLIC_KEY}
    volumes:
      - nginx-cache:/var/cache/nginx
    depends_on:
      - backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

#### Backend Service (Laravel API)
```yaml
# docker-compose.yml for backend
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    ports:
      - "8000:8000"
    environment:
      - APP_ENV=${APP_ENV}
      - APP_URL=${APP_URL}
      - DB_HOST=${RDS_ENDPOINT}
      - DB_DATABASE=${DB_NAME}
      - DB_USERNAME=${DB_USERNAME}
      - DB_PASSWORD=${DB_PASSWORD}
      - REDIS_HOST=${REDIS_ENDPOINT}
      - CACHE_DRIVER=redis
      - SESSION_DRIVER=redis
      - QUEUE_CONNECTION=redis
    volumes:
      - storage-logs:/var/www/storage/logs
      - storage-cache:/var/www/storage/framework/cache
    healthcheck:
      test: ["CMD", "php", "artisan", "tinker", "--execute=echo 'ok';"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### ECS Task Definitions
**Frontend Task Definition:**
```json
{
  "family": "elite-locadora-frontend",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::ACCOUNT:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::ACCOUNT:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "frontend",
      "image": "ACCOUNT.dkr.ecr.REGION.amazonaws.com/elite-locadora-frontend:latest",
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "REACT_APP_API_URL",
          "value": "https://api.elitelocadora.com"
        }
      ],
      "secrets": [
        {
          "name": "REACT_APP_STRIPE_KEY",
          "valueFrom": "arn:aws:ssm:REGION:ACCOUNT:parameter/stripe/public-key"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/elite-locadora-frontend",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost/ || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      }
    }
  ]
}
```

## 🗄️ Database Infrastructure

### Amazon RDS MySQL Configuration
**Production Database Setup:**
```yaml
# RDS Configuration
DBInstanceClass: db.r5.xlarge
Engine: mysql
EngineVersion: '8.0.35'
AllocatedStorage: 100
StorageType: gp3
StorageEncrypted: true
MultiAZ: true
BackupRetentionPeriod: 30
DeletionProtection: true
EnablePerformanceInsights: true
MonitoringInterval: 60

# Database Parameter Group
Parameters:
  innodb_buffer_pool_size: '{DBInstanceClassMemory*3/4}'
  max_connections: 1000
  slow_query_log: 1
  long_query_time: 2
  log_queries_not_using_indexes: 1
  
# Read Replica Configuration
ReadReplicas:
  - InstanceClass: db.r5.large
    Region: us-east-1
    AvailabilityZone: us-east-1b
  - InstanceClass: db.r5.large
    Region: us-west-2
    AvailabilityZone: us-west-2a
```

### Redis Cache Configuration
**ElastiCache Redis Cluster:**
```yaml
# ElastiCache Configuration
CacheNodeType: cache.r6g.large
Engine: redis
EngineVersion: '7.0'
NumCacheNodes: 3
ReplicationGroupDescription: 'Elite Locadora Redis Cluster'
AtRestEncryptionEnabled: true
TransitEncryptionEnabled: true
AuthToken: ${REDIS_AUTH_TOKEN}
SnapshotRetentionLimit: 7
SnapshotWindow: '03:00-05:00'
MaintenanceWindow: 'sun:05:00-sun:06:00'

# Cache Configuration
CacheParameterGroup:
  maxmemory-policy: 'allkeys-lru'
  timeout: 300
  tcp-keepalive: 60
```

## 📁 File Storage & CDN

### Amazon S3 Configuration
**S3 Bucket Structure:**
```
elite-locadora-assets/
├── vehicles/
│   ├── images/
│   │   ├── {vehicle_id}/
│   │   │   ├── main.jpg
│   │   │   ├── interior_1.jpg
│   │   │   └── gallery/
│   └── documents/
│       └── specifications/
├── customers/
│   ├── documents/
│   │   ├── licenses/
│   │   └── contracts/
│   └── profile-images/
├── static/
│   ├── css/
│   ├── js/
│   └── images/
└── backups/
    ├── database/
    └── application/
```

**S3 Bucket Policies:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::elite-locadora-assets/static/*"
    },
    {
      "Sid": "DenyInsecureConnections",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::elite-locadora-assets",
        "arn:aws:s3:::elite-locadora-assets/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

### CloudFront CDN Configuration
```yaml
# CloudFront Distribution
DistributionConfig:
  Origins:
    - DomainName: elite-locadora-assets.s3.amazonaws.com
      Id: S3-elite-locadora-assets
      S3OriginConfig:
        OriginAccessIdentity: origin-access-identity/cloudfront/ABCDEFG1234567
    - DomainName: api.elitelocadora.com
      Id: API-elite-locadora
      CustomOriginConfig:
        HTTPPort: 443
        OriginProtocolPolicy: https-only
        
  DefaultCacheBehavior:
    TargetOriginId: S3-elite-locadora-assets
    ViewerProtocolPolicy: redirect-to-https
    CachePolicyId: 4135ea2d-6df8-44a3-9df3-4b5a84be39ad # CachingOptimized
    OriginRequestPolicyId: 88a5eaf4-2fd4-4709-b370-b4c650ea3fcf # CORS-S3Origin
    
  CacheBehaviors:
    - PathPattern: '/api/*'
      TargetOriginId: API-elite-locadora
      ViewerProtocolPolicy: https-only
      CachePolicyId: 4135ea2d-6df8-44a3-9df3-4b5a84be39ad
      
  Aliases:
    - www.elitelocadora.com
    - elitelocadora.com
    
  ViewerCertificate:
    AcmCertificateArn: arn:aws:acm:us-east-1:ACCOUNT:certificate/CERT-ID
    MinimumProtocolVersion: TLSv1.2_2021
    SslSupportMethod: sni-only
```

## 🔧 Infrastructure as Code (IaC)

### Terraform Configuration
**Main Infrastructure Module:**
```hcl
# main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket         = "elite-locadora-terraform-state"
    key            = "infrastructure/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Project     = "Elite Locadora"
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}

# VPC Configuration
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  name = "elite-locadora-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["${var.aws_region}a", "${var.aws_region}b", "${var.aws_region}c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = true
  enable_vpn_gateway = true
  enable_dns_hostnames = true
  enable_dns_support = true
  
  tags = {
    Terraform = "true"
    Environment = var.environment
  }
}

# ECS Cluster
resource "aws_ecs_cluster" "elite_locadora" {
  name = "elite-locadora-${var.environment}"
  
  capacity_providers = ["FARGATE", "FARGATE_SPOT"]
  
  default_capacity_provider_strategy {
    capacity_provider = "FARGATE"
    weight           = 1
  }
  
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}
```

### Environment Variables Management
**AWS Systems Manager Parameter Store:**
```hcl
# parameter-store.tf
resource "aws_ssm_parameter" "database_url" {
  name  = "/elite-locadora/${var.environment}/database/url"
  type  = "SecureString"
  value = "mysql://${var.db_username}:${var.db_password}@${aws_db_instance.database.endpoint}:3306/${var.db_name}"
  
  tags = {
    Environment = var.environment
  }
}

resource "aws_ssm_parameter" "jwt_secret" {
  name  = "/elite-locadora/${var.environment}/app/jwt-secret"
  type  = "SecureString"
  value = var.jwt_secret
  
  tags = {
    Environment = var.environment
  }
}

resource "aws_ssm_parameter" "stripe_secret_key" {
  name  = "/elite-locadora/${var.environment}/payments/stripe-secret"
  type  = "SecureString"
  value = var.stripe_secret_key
  
  tags = {
    Environment = var.environment
  }
}
```

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow
**.github/workflows/deploy.yml:**
```yaml
name: Deploy Elite Locadora

on:
  push:
    branches: [main, staging, develop]
  pull_request:
    branches: [main]

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY_FRONTEND: elite-locadora-frontend
  ECR_REPOSITORY_BACKEND: elite-locadora-backend

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: password
          MYSQL_DATABASE: elite_locadora_test
        ports:
          - 3306:3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3
      
      redis:
        image: redis:7.0
        ports:
          - 6379:6379
        options: --health-cmd="redis-cli ping" --health-interval=10s --health-timeout=5s --health-retries=3
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
          extensions: dom, curl, libxml, mbstring, zip, pcntl, pdo, sqlite, pdo_sqlite, bcmath, soap, intl, gd, exif, iconv
          coverage: xdebug
          
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
          
      - name: Install Backend Dependencies
        working-directory: backend
        run: |
          composer install --no-progress --prefer-dist --optimize-autoloader
          cp .env.testing .env
          php artisan key:generate
          php artisan migrate --force
          
      - name: Install Frontend Dependencies
        working-directory: frontend
        run: npm ci
        
      - name: Run Backend Tests
        working-directory: backend
        run: |
          php artisan test --coverage --min=80
          
      - name: Run Frontend Tests
        working-directory: frontend
        run: npm run test:ci
        
      - name: Build Frontend
        working-directory: frontend
        run: npm run build

  build-and-deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' || github.ref == 'refs/heads/staging'
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
          
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
        
      - name: Build and push backend image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        run: |
          cd backend
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY_BACKEND:$GITHUB_SHA .
          docker tag $ECR_REGISTRY/$ECR_REPOSITORY_BACKEND:$GITHUB_SHA $ECR_REGISTRY/$ECR_REPOSITORY_BACKEND:latest
          docker push $ECR_REGISTRY/$ECR_REPOSITORY_BACKEND:$GITHUB_SHA
          docker push $ECR_REGISTRY/$ECR_REPOSITORY_BACKEND:latest
          
      - name: Build and push frontend image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        run: |
          cd frontend
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY_FRONTEND:$GITHUB_SHA .
          docker tag $ECR_REGISTRY/$ECR_REPOSITORY_FRONTEND:$GITHUB_SHA $ECR_REGISTRY/$ECR_REPOSITORY_FRONTEND:latest
          docker push $ECR_REGISTRY/$ECR_REPOSITORY_FRONTEND:$GITHUB_SHA
          docker push $ECR_REGISTRY/$ECR_REPOSITORY_FRONTEND:latest
          
      - name: Deploy to ECS
        run: |
          aws ecs update-service --cluster elite-locadora-production --service elite-locadora-backend --force-new-deployment
          aws ecs update-service --cluster elite-locadora-production --service elite-locadora-frontend --force-new-deployment
          
      - name: Wait for deployment
        run: |
          aws ecs wait services-stable --cluster elite-locadora-production --services elite-locadora-backend elite-locadora-frontend
```

## 📊 Monitoring & Logging

### CloudWatch Configuration
**Application Metrics:**
```yaml
# CloudWatch Alarms
Alarms:
  HighCPUUtilization:
    MetricName: CPUUtilization
    Namespace: AWS/ECS
    Threshold: 80
    ComparisonOperator: GreaterThanThreshold
    EvaluationPeriods: 2
    Period: 300
    
  HighMemoryUtilization:
    MetricName: MemoryUtilization
    Namespace: AWS/ECS
    Threshold: 85
    ComparisonOperator: GreaterThanThreshold
    EvaluationPeriods: 2
    Period: 300
    
  DatabaseConnections:
    MetricName: DatabaseConnections
    Namespace: AWS/RDS
    Threshold: 80
    ComparisonOperator: GreaterThanThreshold
    EvaluationPeriods: 3
    Period: 300
    
  APIResponseTime:
    MetricName: TargetResponseTime
    Namespace: AWS/ApplicationELB
    Threshold: 2000
    ComparisonOperator: GreaterThanThreshold
    EvaluationPeriods: 2
    Period: 300
```

### Log Aggregation
**ELK Stack Configuration:**
```yaml
# docker-compose.logging.yml
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
      
  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5044:5044"
    depends_on:
      - elasticsearch
      
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

volumes:
  elasticsearch-data:
```

## 🔐 Security & Compliance

### AWS WAF Configuration
```json
{
  "Name": "elite-locadora-waf",
  "Scope": "CLOUDFRONT",
  "DefaultAction": {
    "Allow": {}
  },
  "Rules": [
    {
      "Name": "AWSManagedRulesCommonRuleSet",
      "Priority": 1,
      "OverrideAction": {
        "None": {}
      },
      "Statement": {
        "ManagedRuleGroupStatement": {
          "VendorName": "AWS",
          "Name": "AWSManagedRulesCommonRuleSet"
        }
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "CommonRuleSetMetric"
      }
    },
    {
      "Name": "RateLimitRule",
      "Priority": 2,
      "Action": {
        "Block": {}
      },
      "Statement": {
        "RateBasedStatement": {
          "Limit": 2000,
          "AggregateKeyType": "IP"
        }
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "RateLimitMetric"
      }
    }
  ]
}
```

### Backup & Disaster Recovery
**Automated Backup Strategy:**
```yaml
# Backup Configuration
DatabaseBackup:
  AutomatedBackups: true
  BackupRetentionPeriod: 30
  BackupWindow: "03:00-04:00"
  CopyTagsToSnapshot: true
  
S3Backup:
  CrossRegionReplication: true
  VersioningEnabled: true
  LifecyclePolicy:
    - Transition:
        Days: 30
        StorageClass: STANDARD_IA
    - Transition:
        Days: 90
        StorageClass: GLACIER
    - Expiration:
        Days: 2555 # 7 years
        
ApplicationBackup:
  ECRImageReplication: true
  ConfigBackup: true
  SecretBackup: true (encrypted)
  CodeBackup: GitHub (primary) + S3 (secondary)
```