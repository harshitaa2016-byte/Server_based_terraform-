# Server-Based Architecture (ECS)

This directory contains a complete ECS Fargate infrastructure with Application Load Balancer.

## 🏗️ Architecture Overview

```
Internet
    ↓
Application Load Balancer (Public Subnets)
    ↓
ECS Fargate Tasks (Private Subnets)
    ↓
CloudWatch Logs
```

## 📋 Components

### Network
- **VPC**: 10.0.0.0/16 with DNS support
- **Public Subnets**: 2 subnets across availability zones
- **Private Subnets**: 2 subnets for ECS tasks
- **Internet Gateway**: For public internet access
- **Route Tables**: Proper routing between subnets

### Security
- **Network ACLs**: Layer 4 security controls
- **Security Groups**: ALB and ECS task security
- **IAM Roles**: ECS execution and task roles

### Compute
- **ECS Cluster**: Fargate cluster with container insights
- **ECS Task Definition**: Container configuration
- **ECS Service**: Manages task lifecycle
- **Application Load Balancer**: HTTP/HTTPS load balancing

### Logging
- **CloudWatch Logs**: Container logging with retention

## 🚀 Quick Start

### 1. Initialize Terraform
```bash
terraform init
```

### 2. Review Plan
```bash
terraform plan
```

### 3. Deploy Infrastructure
```bash
terraform apply
```

### 4. Test Application
```bash
# Get the ALB DNS name
terraform output alb_dns_name

# Test the application
curl http://$(terraform output -raw alb_dns_name)
```

## 📊 Expected Output

After successful deployment, you'll see outputs like:
- `alb_dns_name`: Your application endpoint
- `ecs_cluster_name`: ECS cluster identifier
- `vpc_id`: Network identifier
- `public_subnet_ids`: Load balancer subnets
- `private_subnet_ids`: ECS task subnets

## 🔧 Configuration

### Variables
Key configurable parameters in `variables.tf`:

| Variable | Default | Description |
|----------|---------|-------------|
| `aws_region` | `us-east-1` | AWS region |
| `container_image` | `nginx:latest` | Container image |
| `desired_capacity` | `2` | Number of ECS tasks |
| `vpc_cidr` | `10.0.0.0/16` | VPC CIDR block |

### Customization Examples

#### Change Container Image
```bash
terraform apply -var="container_image=httpd:latest"
```

#### Scale ECS Tasks
```bash
terraform apply -var="desired_capacity=4" -var="max_capacity=8"
```

#### Use Different Region
```bash
terraform apply -var="aws_region=us-west-2"
```

## 🧪 Testing

### Automated Testing
```bash
# Wait for deployment to complete
sleep 300

# Test ALB health
ALB_URL=$(terraform output -raw alb_dns_name)
curl -f http://$ALB_URL || echo "ALB test failed"

# Check ECS service
aws ecs describe-services --cluster $(terraform output -raw ecs_cluster_name) --services $(terraform output -raw ecs_service_name)
```

### Manual Testing
1. Open browser to `http://$(terraform output -raw alb_dns_name)`
2. Should see nginx default page
3. Check CloudWatch logs for container output

## 🧹 Cleanup

```bash
terraform destroy
```

**Note**: This will delete all resources including the ALB and ECS tasks.

## 🔒 Security Considerations

### Network Security
- ECS tasks run in private subnets
- ALB in public subnets with security group restrictions
- Network ACLs provide subnet-level filtering

### IAM Security
- ECS execution role for container registry access
- Task role for application-specific permissions
- Minimal required policies attached

### Best Practices
- Review security group rules before production
- Implement VPC Flow Logs for monitoring
- Use AWS WAF with ALB for web application firewall
- Enable container insights for monitoring

## 🐛 Troubleshooting

### Common Issues

#### ECS Tasks Not Starting
```bash
# Check service events
aws ecs describe-services --cluster CLUSTER_NAME --services SERVICE_NAME

# Check task definition
aws ecs describe-task-definition --task-definition TASK_DEFINITION_NAME
```

#### ALB Not Responding
```bash
# Check target health
aws elbv2 describe-target-health --target-group-arn TARGET_GROUP_ARN

# Check security groups
aws ec2 describe-security-groups --group-ids SG_ID
```

#### Permission Errors
```bash
# Verify IAM roles
aws iam get-role --role-name ROLE_NAME
aws iam list-attached-role-policies --role-name ROLE_NAME
```

## 📈 Monitoring

### CloudWatch Metrics
- ALB request count and latency
- ECS CPU and memory utilization
- Target group health status

### CloudWatch Logs
- Container stdout/stderr logs
- Retention period: 30 days
- Log group: `/ecs/terraform-infrastructure-logs`

## 🔄 Updates

### Update Container Image
```bash
terraform apply -var="container_image=nginx:1.21"
```

### Scale Service
```bash
terraform apply -var="desired_capacity=3"
```

### Update Infrastructure
```bash
terraform plan  # Review changes
terraform apply # Apply updates
```

## 📚 Additional Resources

- [AWS ECS Documentation](https://docs.aws.amazon.com/ecs/)
- [AWS ALB Documentation](https://docs.aws.amazon.com/elasticloadbalancing/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest)
