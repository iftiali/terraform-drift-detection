# Terraform Infrastructure Drift Detection System

[![Terraform](https://img.shields.io/badge/Terraform-1.12.2-623CE4?logo=terraform&logoColor=white)](https://www.terraform.io/)
[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI/CD-2088FF?logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![AWS](https://img.shields.io/badge/AWS-Cloud-FF9900?logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![AWS SES](https://img.shields.io/badge/AWS_SES-Email-FF9900?logo=amazon-aws&logoColor=white)](https://aws.amazon.com/ses/)
[![CloudWatch](https://img.shields.io/badge/CloudWatch-Monitoring-FF9900?logo=amazon-aws&logoColor=white)](https://aws.amazon.com/cloudwatch/)
[![OIDC](https://img.shields.io/badge/OIDC-Authentication-4285F4?logo=openid&logoColor=white)](https://openid.net/connect/)

A comprehensive GitHub Actions-based solution for automated Terraform infrastructure drift detection, notification, and monitoring using AWS services.

## 🎯 Project Overview

This project demonstrates a production-ready infrastructure drift detection system that:
- **Automatically detects** when your AWS infrastructure deviates from its Terraform-defined state
- **Sends email notifications** when drift is detected using AWS SES
- **Logs events** to AWS CloudWatch for monitoring and alerting
- **Uses secure OIDC authentication** to eliminate the need for long-lived AWS credentials
- **Provides multiple deployment workflows** for different infrastructure management scenarios

## 🏗️ Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   GitHub        │    │      AWS         │    │  Notifications  │
│   Actions       │───▶│   Infrastructure │───▶│   & Logging     │
│                 │    │                  │    │                 │
│ • Drift Check   │    │ • S3 Bucket      │    │ • SES Email     │
│ • Plan/Apply    │    │ • OIDC Provider  │    │ • CloudWatch    │
│ • OIDC Setup    │    │ • IAM Roles      │    │ • Artifacts     │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

## 📁 Project Structure

```
terraform_drift/
├── .github/workflows/          # GitHub Actions workflows
│   ├── drift-check.yml        # Main drift detection workflow
│   ├── tfplan-apply.yml       # Infrastructure deployment workflow
│   └── oidc.yml               # OIDC setup workflow
├── oidc/                      # OIDC provider Terraform configuration
│   ├── main.tf               # OIDC provider and IAM role setup
│   ├── locals.tf             # Local values and configurations
│   ├── variables.tf          # Input variables
│   └── output.tf             # Output values
├── s3/                       # Main infrastructure Terraform configuration
│   ├── scripts/              # Python notification and logging scripts
│   │   ├── send_drift_email.py    # Drift detection email notifications
│   │   ├── send_plan_email.py     # Plan review email notifications
│   │   ├── send_outputs_email.py  # Apply success email notifications
│   │   └── log_to_cloudwatch.py   # CloudWatch logging utility
│   ├── main.tf              # S3 bucket infrastructure
│   ├── variables.tf         # Input variables
│   ├── terraform.tf         # Provider and backend configuration
│   └── output.tf            # Output values
└── README.md                # This documentation
```

## 🚀 Features

### 1. Automated Drift Detection
- **Scheduled monitoring**: Configurable cron-based drift checks
- **Manual triggers**: On-demand drift detection via workflow dispatch
- **Smart exit code handling**: Properly distinguishes between no changes (0), errors (1), and drift detected (2)

### 2. Multi-Channel Notifications
- **Email alerts**: Detailed drift reports sent via AWS SES
- **CloudWatch logging**: Centralized logging for monitoring and alerting
- **GitHub artifacts**: Plan files and logs stored as workflow artifacts

### 3. Secure Authentication
- **OIDC integration**: No long-lived AWS credentials required
- **Least privilege**: IAM roles with appropriate permissions
- **GitHub secrets**: Secure storage of sensitive configuration

### 4. Comprehensive Workflows
- **Drift Detection**: Automated infrastructure state monitoring
- **Plan & Apply**: Controlled infrastructure deployment
- **OIDC Setup**: One-time authentication configuration

## 🛠️ Setup Instructions

### Prerequisites
- AWS Account with appropriate permissions
- GitHub repository
- AWS SES configured for email notifications
- Terraform knowledge

### Step 1: Configure AWS SES
1. Set up AWS SES in your preferred region
2. Verify sender and recipient email addresses
3. Note the verified email addresses for configuration

### Step 2: Initial OIDC Setup
1. **Configure GitHub Secrets and Variables**:
   ```
   Secrets:
   - ROLE_ARN: (Will be set after OIDC setup)
   
   Variables:
   - AWS_REGION: your-aws-region (e.g., eu-west-2)
   - SENDER_EMAIL: your-verified-ses-email@domain.com
   - RECIPIENT_EMAIL: recipient@domain.com
   ```

2. **Run OIDC Setup Workflow**:
   - Go to Actions → "oidc-setup"
   - Click "Run workflow"
   - Select "apply"
   - Copy the output Role ARN to GitHub Secrets as `ROLE_ARN`

### Step 3: Deploy Infrastructure
1. **Run Terraform Apply Workflow**:
   - Go to Actions → "terraform: apply and destroy"
   - Click "Run workflow"
   - Select "apply"
   - Review the plan email notification
   - Infrastructure will be deployed automatically

### Step 4: Configure Drift Detection
1. **Enable Scheduled Drift Checks**
   ```yaml
   # In .github/workflows/drift-check.yml
   schedule:
     - cron: "0 9 * * *"  # Daily at 9 AM UTC
   ```

2. **Test Drift Detection**:
   - Manually modify infrastructure in AWS Console
   - Run "Terraform Drift Check" workflow
   - Verify email notification and CloudWatch logs

## 🔧 Configuration

### Environment Variables
| Variable | Description | Example |
|----------|-------------|---------|
| `AWS_REGION` | AWS region for resources | `eu-west-2` |
| `SENDER_EMAIL` | SES verified sender email | `ses@example.com` |
| `RECIPIENT_EMAIL` | Email for notifications | `admin@company.com` |

### Workflow Triggers
| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `drift-check.yml` | Schedule/Manual | Detect infrastructure drift |
| `tfplan-apply.yml` | Manual | Deploy/destroy infrastructure |
| `oidc.yml` | Manual | Setup/teardown OIDC authentication |

## 📊 Monitoring and Alerting

### CloudWatch Integration
- **Log Group**: `/github/terraform/drift`
- **Log Streams**: Daily streams with drift detection results
- **Retention**: Configure as needed for compliance

### Email Notifications
- **Drift Detected**: Includes full Terraform plan as attachment
- **Plan Review**: Sent before infrastructure changes
- **Apply Success**: Confirmation with infrastructure outputs

## 🔍 Troubleshooting

### Common Issues

1. **Exit Code 0 Instead of 2 for Drift**:
   - Ensure `terraform_wrapper: false` in setup-terraform action
   - Verify `detailed-exitcode` flag is used in terraform plan

2. **Email Sending Failures**:
   - Verify SES email addresses are verified
   - Check IAM permissions for SES actions
   - Ensure AWS region matches SES configuration

3. **OIDC Authentication Errors**:
   - Verify GitHub repository conditions in OIDC trust policy
   - Check that `ROLE_ARN` secret is correctly set
   - Ensure IAM role has necessary permissions

4. **Script Path Errors**:
   - Verify working directory is set to `./s3`
   - Check that Python scripts exist in `scripts/` subdirectory
   - Ensure scripts have proper file permissions

## 🔐 Security Considerations

- **OIDC Authentication**: Eliminates need for long-lived AWS credentials
- **Least Privilege**: IAM roles have minimal required permissions
- **Encrypted Storage**: Terraform state stored in encrypted S3 bucket
- **Access Controls**: S3 bucket blocks all public access
