# Keycloak Installation Steps

Follow these steps to install Keycloak in your AWS account.

## Prerequisites

- AWS CLI configured with `datahub-rep` profile
- Docker installed locally
- All existing CloudFormation stacks deployed (Networking, RDS, LoadBalancer, ECS, etc.)
- Access to edit CloudFormation templates

## Step 1: Update CloudFormation Templates

### 1.1 Update `modules/ECR.yaml`

Add the Keycloak repository after line 81 (after `UserServiceRepository`):

```yaml
    KeycloakRepository:
        Type: "AWS::ECR::Repository"
        Properties:
            RepositoryName: !Sub "${ProjectName}-keycloak/${Environment}"
```

### 1.2 Update `modules/LoadBalancer.yaml`

**Add Target Group** (before Outputs section, around line 1207):

```yaml
    KeycloakTargetGroup:
        Type: "AWS::ElasticLoadBalancingV2::TargetGroup"
        Properties:
            HealthCheckIntervalSeconds: 30
            HealthCheckPath: "/health/ready"
            Port: 8080
            Protocol: "HTTP"
            HealthCheckPort: "traffic-port"
            HealthCheckProtocol: "HTTP"
            HealthCheckTimeoutSeconds: 5
            UnhealthyThresholdCount: 2
            TargetType: "ip"
            Matcher:
                HttpCode: "200"
            HealthyThresholdCount: 5
            VpcId:
              Fn::ImportValue: !Sub ${ProjectName}-Networking-${Environment}-VPC-ID
            Name: !Sub "${ProjectName}-Keycloak-${Environment}"
            HealthCheckEnabled: true
            TargetGroupAttributes:
              -
                Key: "target_group_health.unhealthy_state_routing.minimum_healthy_targets.percentage"
                Value: "off"
              -
                Key: "deregistration_delay.timeout_seconds"
                Value: "300"
              -
                Key: "stickiness.type"
                Value: "lb_cookie"
              -
                Key: "stickiness.lb_cookie.duration_seconds"
                Value: "86400"
              -
                Key: "stickiness.enabled"
                Value: "true"
              -
                Key: "load_balancing.algorithm.type"
                Value: "round_robin"
```

**Add Listener Rule** (after `ApprovedDataServiceListenerRule`, around line 338):

```yaml
    KeycloakListenerRule:
        Type: "AWS::ElasticLoadBalancingV2::ListenerRule"
        Properties:
            Priority: "10"
            ListenerArn: !Ref HttpsLoadBalancerListener
            Conditions:
              -
                Field: "path-pattern"
                PathPatternConfig:
                    Values:
                      - "/auth*"
                      - "/realms*"
            Actions:
              -
                Type: "forward"
                TargetGroupArn: !Ref KeycloakTargetGroup
                ForwardConfig:
                    TargetGroups:
                      -
                        TargetGroupArn: !Ref KeycloakTargetGroup
                        Weight: 1
                    TargetGroupStickinessConfig:
                        Enabled: true
                        DurationSeconds: 86400
```

**Add Export in Outputs section** (after `UserServiceTargetGroupArn`, around line 1257):

```yaml
  KeycloakTargetGroupArn:
    Value: !Ref KeycloakTargetGroup
    Export:
      Name: !Sub "${AWS::StackName}-KeycloakTargetGroup"
```

### 1.3 Update `modules/ECS.yaml`

**Add Keycloak Service** (after `SubmissionService`, around line 321):

```yaml
    KeycloakService:
        Type: "AWS::ECS::Service"
        Properties:
            ServiceName: !Sub "${ProjectName}-Keycloak"
            Cluster: !GetAtt ECSCluster.Arn
            LoadBalancers:
              -
                TargetGroupArn:
                  Fn::ImportValue: !Sub "${ProjectName}-LoadBalancer-${Environment}-KeycloakTargetGroup"
                ContainerName: !Sub "${ProjectName}-Keycloak"
                ContainerPort: 8080
            DesiredCount: 1
            PlatformVersion: "LATEST"
            TaskDefinition: !Ref KeycloakTaskDefinition
            DeploymentConfiguration:
                MaximumPercent: 200
                MinimumHealthyPercent: 100
                DeploymentCircuitBreaker:
                    Enable: true
                    Rollback: true
            NetworkConfiguration:
                AwsvpcConfiguration:
                    AssignPublicIp: "DISABLED"
                    SecurityGroups:
                      - Fn::ImportValue: !Sub ${ProjectName}-Networking-${Environment}-ECS-Host-SecurityGroup
                    Subnets:
                      - Fn::ImportValue: !Sub "${ProjectName}-Networking-${Environment}-Private-Subnet-AZ1"
                      - Fn::ImportValue: !Sub "${ProjectName}-Networking-${Environment}-Private-Subnet-AZ2"
            HealthCheckGracePeriodSeconds: 300
            SchedulingStrategy: "REPLICA"
            DeploymentController:
                Type: "ECS"
            CapacityProviderStrategy:
              -
                CapacityProvider: "FARGATE"
                Weight: 0
                Base: 0
              -
                CapacityProvider: "FARGATE_SPOT"
                Weight: 1
                Base: 0
```

**Add Keycloak Task Definition** (after `DownloadServiceTaskDefinition`, around line 651):

```yaml
    KeycloakTaskDefinition:
        Type: "AWS::ECS::TaskDefinition"
        Properties:
            ContainerDefinitions:
              -
                Cpu: 2048
                Environment:
                  -
                    Name: "KEYCLOAK_ADMIN"
                    Value: "admin"
                  -
                    Name: "KC_DB"
                    Value: "dev-file"
                  -
                    Name: "KC_HOSTNAME_STRICT"
                    Value: "false"
                  -
                    Name: "KC_HOSTNAME_STRICT_HTTPS"
                    Value: "false"
                  -
                    Name: "KC_HTTP_ENABLED"
                    Value: "true"
                  -
                    Name: "KC_HEALTH_ENABLED"
                    Value: "true"
                Secrets:
                  -
                    Name: "KEYCLOAK_ADMIN_PASSWORD"
                    ValueFrom: !Sub "arn:aws:secretsmanager:${AWS::Region}:${AWS::AccountId}:secret:${ProjectName}_application_${Environment}:KEYCLOAK_ADMIN_PASSWORD::"
                Essential: true
                HealthCheck:
                    Command:
                      - "CMD-SHELL"
                      - "curl -f http://localhost:8080/health/ready || exit 1"
                    Interval: 30
                    Timeout: 5
                    Retries: 3
                    StartPeriod: 300
                Image: !Sub "${AWS::AccountId}.dkr.ecr.${AWS::Region}.amazonaws.com/${ProjectName}-keycloak/${Environment}:latest"
                LogConfiguration:
                    LogDriver: "awslogs"
                    Options:
                        awslogs-group: !Sub "/aws/ecs/${ProjectName}-microservices-${Environment}"
                        awslogs-region: !Ref AWS::Region
                        awslogs-stream-prefix: "keycloak"
                Memory: 4096
                Name: !Sub "${ProjectName}-Keycloak"
                PortMappings:
                  -
                    ContainerPort: 8080
                    HostPort: 8080
                    Protocol: "tcp"
            Family: !Sub "${ProjectName}-keycloak-${Environment}"
            TaskRoleArn: !Ref ECSTaskRole
            ExecutionRoleArn: !Ref ECSExecutionRole
            NetworkMode: "awsvpc"
            RequiresCompatibilities:
              - "EC2"
              - "FARGATE"
            Cpu: "2048"
            Memory: "4096"
```

### 1.4 Update `modules/SecretsManager.yaml`

Find the `ApplicationSecrets` resource and add to the SecretString JSON:

```json
"KEYCLOAK_ADMIN_PASSWORD": "REPLACEME"
```

Replace `REPLACEME` with a strong password for the Keycloak admin user.

## Step 2: Deploy CloudFormation Stacks

Deploy stacks in this order:

```bash
# Set your environment and project name
ENV=dev  # or test, prod
PROJECT_NAME=redwood  # or your project name

# 1. Deploy ECR stack (creates Keycloak repository)
cd /Users/atti/DATAHUB/datahub-cloud-replication
aws cloudformation deploy \
  --stack-name DataHub-ECR-${ENV} \
  --template-file modules/ECR.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1 \
  --profile datahub-rep

# 2. Deploy LoadBalancer stack (creates target group and listener rule)
aws cloudformation deploy \
  --stack-name DataHub-LoadBalancer-${ENV} \
  --template-file modules/LoadBalancer.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1 \
  --profile datahub-rep

# 3. Deploy SecretsManager stack (adds Keycloak admin password)
aws cloudformation deploy \
  --stack-name DataHub-SecretsManager-${ENV} \
  --template-file modules/SecretsManager.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1 \
  --profile datahub-rep

# 4. Deploy ECS stack (creates Keycloak service and task definition)
aws cloudformation deploy \
  --stack-name DataHub-ECS-${ENV} \
  --template-file modules/ECS.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1 \
  --profile datahub-rep
```

## Step 3: Build and Push Keycloak Docker Image

Use the deployment script:

```bash
cd /Users/atti/DATAHUB/datahub-deployment-scripts
python deploy.py ${PROJECT_NAME} keycloak ${ENV} latest
```

This will:
- Build the Keycloak Docker image
- Push it to ECR
- Trigger ECS deployment

## Step 4: Verify Deployment

### Check ECS Service Status

```bash
aws ecs describe-services \
  --cluster ${PROJECT_NAME}-Services-${ENV} \
  --services ${PROJECT_NAME}-Keycloak \
  --region us-east-1 \
  --profile datahub-rep
```

### Check Target Group Health

```bash
# Get Load Balancer DNS name
LB_DNS=$(aws cloudformation describe-stacks \
  --stack-name DataHub-LoadBalancer-${ENV} \
  --query 'Stacks[0].Outputs[?OutputKey==`LoadBalancerDNSName`].OutputValue' \
  --output text \
  --region us-east-1 \
  --profile datahub-rep)

echo "Load Balancer DNS: ${LB_DNS}"
```

### Access Keycloak

- **Keycloak Admin Console**: `http://${LB_DNS}/auth/admin`
- **Keycloak Welcome**: `http://${LB_DNS}/auth`
- **Default Admin Username**: `admin`
- **Default Admin Password**: Value from Secrets Manager (`KEYCLOAK_ADMIN_PASSWORD`)

## Troubleshooting

### Service Not Starting
- Check ECS task logs in CloudWatch: `/aws/ecs/${PROJECT_NAME}-microservices-${ENV}`
- Verify admin password secret exists in Secrets Manager
- Check container logs for startup errors

### Health Check Failing
- Verify health endpoint: `curl http://localhost:8080/health/ready` (from within container)
- Check that container has sufficient memory (4GB configured)
- Review CloudWatch logs for errors

### Cannot Access Keycloak
- Verify Load Balancer listener rules are correct
- Check security group rules allow traffic on port 8080
- Verify DNS/Route53 configuration if using custom domain

## Next Steps

1. Log into Keycloak Admin Console
2. Create a realm for your application
3. Configure clients and users
4. Set up identity providers if needed
5. Integrate with your DataHub services
