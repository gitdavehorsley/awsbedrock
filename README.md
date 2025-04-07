# AWS Bedrock Security Controls

This repository contains CloudFormation templates that implement comprehensive security controls for AWS Bedrock. The implementation is split into two templates for better modularity and separation of concerns.

## Architecture Overview

The security controls are implemented in two layers:

1. **Network Infrastructure** (`bedrock-network.yaml`)
   - VPC Endpoint for private Bedrock access
   - Security Group with restricted network traffic
   - DNS configuration for VPC endpoint

2. **Security Controls** (`bedrock-controls.yaml`)
   - IAM roles and policies with least privilege access
   - CloudWatch logging for monitoring
   - SNS notifications for alerts
   - Budget controls for cost management
   - Native Bedrock guardrails for content filtering and security

## Prerequisites

- AWS CLI installed and configured
- Existing VPC and subnet
- Permissions to create:
  - VPC Endpoints
  - Security Groups
  - IAM Roles and Policies
  - CloudWatch Log Groups
  - SNS Topics
  - AWS Budgets
  - Bedrock Guardrails

## Deployment

### Step 1: Deploy Network Infrastructure

First, deploy the network infrastructure that enables private access to Bedrock:

```bash
aws cloudformation create-stack \
  --stack-name bedrock-network \
  --template-body file://bedrock-network.yaml \
  --parameters \
    ParameterKey=ExistingVpcId,ParameterValue=<your-vpc-id> \
    ParameterKey=ExistingSubnetId,ParameterValue=<your-subnet-id> \
    ParameterKey=EnvironmentName,ParameterValue=<environment> \
    ParameterKey=VpcCidr,ParameterValue=<vpc-cidr>
```

Replace the following values:
- `<your-vpc-id>`: ID of your existing VPC
- `<your-subnet-id>`: ID of your existing subnet
- `<environment>`: Environment name (e.g., dev, test, prod)
- `<vpc-cidr>`: CIDR range for VPC ingress rule (e.g., 10.0.0.0/16)

### Step 2: Deploy Security Controls

Once the network infrastructure is deployed, deploy the security controls:

```bash
aws cloudformation create-stack \
  --stack-name bedrock-controls \
  --template-body file://bedrock-controls.yaml \
  --parameters \
    ParameterKey=AdminEmail,ParameterValue=<your-email> \
    ParameterKey=MonthlyBudgetAmount,ParameterValue=<budget-amount> \
    ParameterKey=EnvironmentName,ParameterValue=<environment> \
    ParameterKey=NetworkStackName,ParameterValue=bedrock-network \
    ParameterKey=GuardrailName,ParameterValue=<guardrail-name> \
  --capabilities CAPABILITY_NAMED_IAM
```

Replace the following values:
- `<your-email>`: Email address for notifications
- `<budget-amount>`: Monthly budget limit in USD
- `<environment>`: Environment name (same as network stack)
- `<guardrail-name>`: Name for the Bedrock guardrail (defaults to bedrock-guardrail-{environment})

## Template Details

### Network Infrastructure Template (`bedrock-network.yaml`)

This template creates:
- Security group for Bedrock endpoint with:
  - Inbound access on port 443 from VPC CIDR
  - All outbound access allowed
- VPC Endpoint for Bedrock runtime with:
  - Interface endpoint type
  - Private DNS disabled (to avoid conflicts)
  - Placement in specified subnet

### Security Controls Template (`bedrock-controls.yaml`)

This template creates:
- IAM role and policy for Bedrock access with:
  - Least privilege permissions
  - Access to specific foundation models
  - Environment-based access control
- CloudWatch log group for model invocations
- SNS topic for alerts and notifications
- Budget controls with email notifications
- Native Bedrock guardrails with:
  - Input content filtering
  - Blocked terms and categories
  - Security and privacy controls
  - Prevention policies for harmful content

## Outputs

### Network Stack Outputs
- `BedrockSecurityGroupId`: ID of the created security group
- `BedrockVPCEndpointId`: ID of the VPC endpoint
- `BedrockVPCEndpointDNS`: DNS entries for the VPC endpoint

### Controls Stack Outputs
- `BedrockAccessRoleARN`: ARN of the Bedrock access role
- `BedrockAlertsTopicARN`: ARN of the SNS topic for alerts
- `BedrockGuardrailId`: ID of the created Bedrock guardrail
- `BedrockGuardrailVersionId`: ID of the active guardrail version

## Security Considerations

1. **Network Security**
   - Private VPC endpoint prevents Bedrock traffic from traversing the internet
   - Restricted security group controls network access

2. **Access Control**
   - Least privilege IAM permissions
   - Environment-based access control
   - Limited to specific foundation models
   - Content filtering guardrails

3. **Content Security**
   - Input filtering for sensitive terms
   - Blocked categories (hate speech, profanity, etc.)
   - Prevention of malicious code generation
   - Protection against LLM jailbreaking

4. **Monitoring and Alerting**
   - CloudWatch logging for audit trail
   - SNS notifications for important events
   - Budget alerts for cost control

## Maintenance

1. **Updating Templates**
   - Use CloudFormation change sets to preview changes
   - Update network and controls stacks independently
   - Test changes in non-production environment first

2. **Monitoring**
   - Review CloudWatch logs regularly
   - Monitor budget utilization
   - Check SNS notifications

## Troubleshooting

1. **VPC Endpoint Issues**
   - Verify DNS settings in VPC
   - Check security group rules
   - Validate subnet routing

2. **IAM Issues**
   - Verify role trust relationships
   - Check policy permissions
   - Validate environment tags

3. **Guardrail Issues**
   - Verify guardrail activation status
   - Check blocked terms and categories
   - Review prevention policies

4. **Budget/Alerting Issues**
   - Confirm SNS topic subscriptions
   - Verify email addresses
   - Check CloudWatch logs

## Contributing

Feel free to submit issues and enhancement requests!
