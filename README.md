# AWS Bedrock Security Controls

This repository contains a comprehensive CloudFormation template that implements security controls for AWS Bedrock, including network infrastructure, IAM roles, monitoring, and native guardrails. The template has been designed following the **OWASP Top 10 for LLM Applications** security framework to address AI-specific vulnerabilities.

## Architecture Overview

The security controls are implemented in a single template with multiple components:

1. **Network Infrastructure**
   - VPC Endpoint for private Bedrock access
   - Security Group with restricted network traffic
   - DNS configuration for VPC endpoint

2. **Security Controls**
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

Deploy the complete stack using the following command:

```bash
aws cloudformation create-stack \
  --stack-name bedrock-security \
  --template-body file://BedrockControls.yaml \
  --parameters \
    ParameterKey=ExistingVpcId,ParameterValue=<your-vpc-id> \
    ParameterKey=ExistingSubnetId,ParameterValue=<your-subnet-id> \
    ParameterKey=VpcCidr,ParameterValue=<vpc-cidr> \
    ParameterKey=AdminEmail,ParameterValue=<your-email> \
    ParameterKey=MonthlyBudgetAmount,ParameterValue=<budget-amount> \
    ParameterKey=EnvironmentName,ParameterValue=<environment> \
    ParameterKey=GuardrailName,ParameterValue=<guardrail-name> \
  --capabilities CAPABILITY_NAMED_IAM
```

Replace the following values:
- `<your-vpc-id>`: ID of your existing VPC
- `<your-subnet-id>`: ID of your existing subnet
- `<vpc-cidr>`: CIDR range for VPC ingress rule (e.g., 10.0.0.0/16)
- `<your-email>`: Email address for notifications
- `<budget-amount>`: Monthly budget limit in USD
- `<environment>`: Environment name (e.g., dev, test, prod)
- `<guardrail-name>`: Name for the Bedrock guardrail

## Template Details

The template (`BedrockControls.yaml`) creates:

1. **Network Infrastructure**
   - Security group for Bedrock endpoint with restricted access (port 443 from VPC CIDR only)
   - VPC Endpoint for Bedrock runtime with private DNS disabled
   - Policy-based endpoint access control

2. **IAM and Security Controls**
   - IAM role and policy for Bedrock access with least privilege permissions
   - Access restricted to specific foundation models and regions
   - CloudWatch log group for model invocations with 30-day retention
   - SNS topic for alerts and notifications

3. **Budget Controls**
   - AWS Budget with cost alerts at 80% (actual) and 100% (forecasted)
   - Email notifications for budget thresholds

4. **Bedrock Guardrails**
   - Content policy configuration for hate, sexual, violence, and misconduct filtering
   - Word policy with profanity filtering and security-related term blocking
   - PII detection and blocking/anonymization for emails, phones, SSNs, credit cards
   - Regex-based API key and token detection

## OWASP LLM Top 10 Security Mitigations

This template addresses the OWASP Top 10 for LLM Applications vulnerabilities:

### LLM01: Prompt Injection
**How the template mitigates:**
- **Bedrock Guardrails**: Filters malicious inputs and blocks security-related terms like "hack", "exploit", "vulnerability"
- **Input Sanitization**: Guardrails detect and block potentially harmful prompts
- **Least Privilege Access**: IAM policies restrict model access and prevent unauthorized backend access

**Additional recommendations:**
- Implement human-in-the-loop approval for sensitive operations
- Use separate user contexts and avoid mixing system prompts with user input
- Consider implementing output validation in your application layer

### LLM02: Insecure Output Handling
**How the template mitigates:**
- **Output Monitoring**: CloudWatch logging captures all model invocations for review
- **Content Filtering**: Guardrails block inappropriate output content
- **Alerting**: SNS notifications for unusual activity patterns

**Additional recommendations:**
- Never pass LLM output directly to system shells or eval functions
- Implement proper output encoding for web applications
- Use content validation before processing LLM responses

### LLM03: Training Data Poisoning
**How the template mitigates:**
- **Managed Models**: Uses AWS Bedrock foundation models with vetted training data
- **Model Restrictions**: IAM policies limit access to approved, maintained models
- **Supply Chain Security**: Restricts to AWS-managed model endpoints

**Additional recommendations:**
- If fine-tuning models, implement strict data validation pipelines
- Monitor model outputs for bias or unexpected behavior
- Use version-controlled, audited training datasets

### LLM04: Model Denial of Service
**How the template mitigates:**
- **Rate Limiting**: VPC endpoint policies can implement request throttling
- **Resource Monitoring**: CloudWatch alarms monitor API usage patterns
- **Budget Controls**: Cost monitoring prevents runaway usage

**Additional recommendations:**
- Implement API rate limiting in your application
- Use input length validation to prevent context window overflow
- Monitor for unusual query patterns or resource consumption

### LLM05: Supply Chain Vulnerabilities
**How the template mitigates:**
- **Managed Service**: Uses AWS Bedrock managed models, reducing supply chain risk
- **Access Controls**: Restricts model access to approved versions and regions
- **Infrastructure Security**: Private VPC endpoints prevent unauthorized access

**Additional recommendations:**
- Regularly review and update model versions
- Implement security scanning for any custom models or plugins
- Maintain an inventory of all AI components and dependencies

### LLM06: Sensitive Information Disclosure
**How the template mitigates:**
- **PII Detection**: Guardrails automatically detect and block/anonymize sensitive data
- **Data Loss Prevention**: Blocks emails, phone numbers, SSNs, credit card numbers
- **API Key Protection**: Regex patterns detect and block API keys and tokens
- **Logging Controls**: Structured logging with data retention policies

**Additional recommendations:**
- Implement data classification and handling policies
- Use data masking for sensitive information in logs
- Regular audit logs for potential data leakage

### LLM07: Insecure Plugin Design
**How the template mitigates:**
- **No Plugin Support**: Template focuses on core Bedrock models without plugins
- **Restricted Access**: IAM policies limit available model functions
- **Network Isolation**: Private VPC endpoints control external access

**Additional recommendations:**
- If using plugins, implement strict input validation
- Use least privilege access for plugin operations
- Implement plugin sandboxing and security reviews

### LLM08: Excessive Agency
**How the template mitigates:**
- **Limited Permissions**: IAM policies restrict actions to model invocation only
- **No Administrative Access**: Models cannot perform system administration tasks
- **Audit Trail**: All actions are logged for review

**Additional recommendations:**
- Implement human approval workflows for high-risk operations
- Use role-based access control for different user types
- Limit model access to only necessary functions

### LLM09: Overreliance
**How the template mitigates:**
- **Monitoring and Alerting**: CloudWatch metrics track model usage patterns
- **Output Filtering**: Guardrails help identify potentially unreliable content
- **Audit Capabilities**: Comprehensive logging enables output review

**Additional recommendations:**
- Implement output verification mechanisms
- Provide user education about model limitations
- Use confidence scoring and human review for critical decisions

### LLM10: Model Theft
**How the template mitigates:**
- **Network Security**: Private VPC endpoints prevent unauthorized access
- **Access Controls**: Strong IAM policies limit who can access models
- **Audit Logging**: CloudWatch logs track all access attempts
- **AWS Managed Models**: Uses AWS-hosted models, reducing theft risk

**Additional recommendations:**
- Implement API rate limiting to prevent model extraction
- Monitor for unusual query patterns that might indicate theft attempts
- Use watermarking for custom model outputs

## Security Considerations

1. **Network Security**
   - Private VPC endpoint prevents Bedrock traffic from traversing the internet
   - Security group restricts access to HTTPS (port 443) from VPC CIDR only
   - No public internet access to Bedrock APIs

2. **Access Control**
   - IAM least privilege permissions limited to specific models and actions
   - Environment-based tagging for conditional access
   - No cross-account access without explicit permissions

3. **Content Security**
   - Multi-layered content filtering (hate speech, profanity, misconduct, violence, sexual content)
   - PII detection and blocking for compliance
   - Security-related term filtering to prevent exploitation attempts
   - Custom regex patterns for API key/token detection

4. **Monitoring and Alerting**
   - Comprehensive CloudWatch logging for audit trail
   - Real-time SNS notifications for security events
   - Budget monitoring with cost alerts
   - High usage alarms for anomaly detection

5. **Data Protection**
   - 30-day log retention for compliance balance
   - PII anonymization where possible
   - Encrypted data in transit via HTTPS
   - No sensitive data stored in template parameters

## Outputs

The template provides the following outputs:

**Network Outputs:**
- `BedrockSecurityGroupId`: Security group for Bedrock endpoint
- `BedrockVPCEndpointId`: VPC endpoint for private access
- `BedrockVPCEndpointDNS`: DNS configuration for applications

**Security Control Outputs:**
- `BedrockAccessRoleARN`: IAM role for least privilege access
- `BedrockRestrictedPolicyARN`: Managed policy for controlled permissions
- `BedrockAlertsTopicARN`: SNS topic for security notifications
- `BedrockLogGroupName`: CloudWatch log group for auditing

**Guardrail Outputs:**
- `BedrockGuardrailId`: Guardrail identifier for application integration
- `BedrockGuardrailArn`: Guardrail ARN for cross-service references
- `BedrockGuardrailVersion`: Active version for change tracking

## Maintenance and Best Practices

### Regular Security Reviews
1. **Monthly Reviews:**
   - Review CloudWatch logs for unusual patterns
   - Analyze budget usage and cost trends
   - Check SNS alerts for security events
   - Update guardrail policies based on new threats

2. **Quarterly Reviews:**
   - Review and update IAM policies
   - Test alert mechanisms and response procedures
   - Update model access permissions based on business needs
   - Review and update blocked word lists

### Monitoring Best Practices
1. **Set up automated dashboards** for key security metrics
2. **Implement log aggregation** for security event correlation
3. **Configure automated responses** for critical security alerts
4. **Regular penetration testing** of LLM applications

### Compliance Considerations
- **Data Residency**: Models run in specified AWS region only
- **Audit Trail**: Comprehensive logging for compliance requirements
- **Data Retention**: Configurable log retention periods
- **Access Control**: Role-based access with audit capabilities

## Troubleshooting

### Common Issues

1. **VPC Endpoint Connectivity:**
   - Verify DNS settings in VPC configuration
   - Check security group rules allow HTTPS traffic
   - Confirm subnet routing tables

2. **IAM Permission Errors:**
   - Verify user/role has Environment tag
   - Check policy attachments and trust relationships
   - Confirm model ARNs in policy match intended models

3. **Guardrail Issues:**
   - Verify guardrail status is "READY"
   - Check blocked terms don't interfere with legitimate use cases
   - Review PII anonymization settings

4. **Budget/Alerting:**
   - Confirm SNS topic subscriptions
   - Check email addresses in notifications
   - Verify CloudWatch alarm thresholds

## Contributing

When contributing security improvements:

1. **Test all changes** in non-production environment
2. **Document security implications** of modifications
3. **Update this README** with new security considerations
4. **Follow AWS security best practices**
5. **Consider OWASP LLM Top 10** implications for changes

## License

This template is provided as-is for educational and implementation purposes. Users are responsible for ensuring compliance with their organization's security policies and regulatory requirements.

## Additional Resources

- [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [AWS Bedrock Security Best Practices](https://docs.aws.amazon.com/bedrock/latest/userguide/security.html)
- [AWS Well-Architected Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/)
- [AWS VPC Endpoint Security](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-access.html)
