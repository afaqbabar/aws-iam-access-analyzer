# 🕵️‍♂️ AWS IAM Access Analyzer - Hands-On Lab

[![AWS](https://img.shields.io/badge/AWS-Free%20Tier-orange?style=flat-square&logo=amazon-aws)](https://aws.amazon.com/free/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

> **Detect and prevent accidental public access in AWS - 100% Free**

A practical, hands-on guide to setting up and using AWS IAM Access Analyzer to identify unintended resource exposure. Perfect for learning AWS security best practices without spending a dime.

![IAM Access Analyzer Demo](https://via.placeholder.com/800x400/232F3E/FF9900?text=AWS+IAM+Access+Analyzer+Demo)

---

## 📋 Table of Contents

- [Overview](#-overview)
- [What You'll Learn](#-what-youll-learn)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Step-by-Step Guide](#-step-by-step-guide)
- [Architecture](#-architecture)
- [Cost Breakdown](#-cost-breakdown)
- [Best Practices](#-best-practices)
- [Troubleshooting](#-troubleshooting)
- [Additional Resources](#-additional-resources)
- [Contributing](#-contributing)
- [License](#-license)
- [Author](#-author)

---

## 🎯 Overview

Accidental public access is one of the most common security misconfigurations in AWS. **IAM Access Analyzer** helps you detect and prevent it automatically - and it's **100% free**.

### Why This Matters

- 🚨 Misconfigured S3 buckets have led to major data breaches
- 💰 Security incidents can cost millions in damages and fines
- ⚡ Access Analyzer provides continuous, automated monitoring
- 🆓 Completely free with no usage limits

### What is IAM Access Analyzer?

IAM Access Analyzer uses automated reasoning to analyze resource-based policies and identify resources shared with external entities. It continuously monitors your AWS resources and alerts you to:

- **Public access** - Resources accessible by anyone on the internet
- **Cross-account access** - Resources shared with other AWS accounts  
- **Cross-organization access** - Resources shared outside your AWS Organization

---

## 🎓 What You'll Learn

By completing this lab, you'll gain hands-on experience with:

- ✅ Enabling IAM Access Analyzer for your AWS account
- ✅ Creating and configuring S3 buckets
- ✅ Understanding S3 bucket policies and public access controls
- ✅ Detecting public resource exposure automatically
- ✅ Remediating security findings
- ✅ Validating IAM policies before deployment
- ✅ AWS CLI operations and best practices

**Time Required:** ~20 minutes  
**Difficulty Level:** Beginner to Intermediate  
**Cost:** $0.00 (AWS Free Tier)

---

## 🔧 Prerequisites

### Required

- **AWS Account** - [Sign up for free](https://aws.amazon.com/free/)
- **AWS CLI v2** - [Installation guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- **IAM Permissions:**
  - `access-analyzer:*`
  - `s3:*`

### Recommended

- Basic understanding of AWS IAM concepts
- Familiarity with JSON syntax
- Terminal/Command line experience

### Setup

1. **Install AWS CLI v2**
   ```bash
   # macOS
   brew install awscli
   
   # Windows
   # Download from: https://awscli.amazonaws.com/AWSCLIV2.msi
   
   # Linux
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install
   ```

2. **Configure AWS CLI**
   ```bash
   aws configure
   # Enter your AWS Access Key ID
   # Enter your AWS Secret Access Key
   # Enter your default region (e.g., eu-central-1)
   # Enter your default output format (json)
   ```

3. **Verify Installation**
   ```bash
   aws --version
   # Should output: aws-cli/2.x.x ...
   ```

---

## 🚀 Quick Start

For those who want to dive right in:

```bash
# 1. Clone the repository
git clone https://github.com/afaqbabar/aws-iam-access-analyzer-lab.git
cd aws-iam-access-analyzer-lab

# 2. Create the analyzer
aws accessanalyzer create-analyzer \
  --analyzer-name MyFreeAnalyzer \
  --type ACCOUNT

# 3. Run the demo script
chmod +x demo.sh
./demo.sh

# 4. Check findings (wait ~2 minutes)
aws accessanalyzer list-findings \
  --analyzer-arn $(aws accessanalyzer list-analyzers --query 'analyzers[0].arn' --output text)

# 5. Clean up
./cleanup.sh
```

---

## 📖 Step-by-Step Guide

### Step 1: Create an IAM Access Analyzer

**Option A: AWS Console**

1. Navigate to [IAM Console](https://console.aws.amazon.com/iam/)
2. Select **Access Analyzer** from the left menu
3. Click **Create analyzer**
4. Configure:
   - **Name:** `MyFreeAnalyzer`
   - **Type:** Account
   - **Region:** Your preferred region
5. Click **Create**

**Option B: AWS CLI**

```bash
aws accessanalyzer create-analyzer \
  --analyzer-name MyFreeAnalyzer \
  --type ACCOUNT
```

**Expected Output:**
```json
{
    "arn": "arn:aws:access-analyzer:eu-central-1:123456789012:analyzer/MyFreeAnalyzer"
}
```

---

### Step 2: Create a Test S3 Bucket

Create an S3 bucket that we'll intentionally misconfigure to generate a finding.

```bash
aws s3api create-bucket \
  --bucket my-public-demo-bucket-demo123 \
  --region eu-central-1 \
  --create-bucket-configuration LocationConstraint=eu-central-1
```

> **Note:** Bucket names must be globally unique. Change `demo123` to something unique if needed.

> **Windows PowerShell Users:** Replace backslashes (`\`) with backticks (`` ` ``) for line continuation.

**Expected Output:**
```json
{
    "Location": "http://my-public-demo-bucket-demo123.s3.amazonaws.com/"
}
```

---

### Step 3: Disable Block Public Access

By default, S3 blocks all public access. We'll temporarily disable this for demonstration purposes.

```bash
aws s3api put-public-access-block \
  --bucket my-public-demo-bucket-demo123 \
  --public-access-block-configuration \
    "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"
```

> ⚠️ **Security Warning:** In production environments, always keep Block Public Access enabled unless you have a documented business requirement.

---

### Step 4: Apply a Public Bucket Policy

Create a file named `bucket-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-public-demo-bucket-demo123/*"
    }
  ]
}
```

Apply the policy:

```bash
aws s3api put-bucket-policy \
  --bucket my-public-demo-bucket-demo123 \
  --policy file://bucket-policy.json
```

**What this does:** Grants public read access to all objects in the bucket.

---

### Step 5: View Access Analyzer Findings

Wait approximately **2 minutes** for Access Analyzer to detect the public access.

**List your analyzers:**
```bash
aws accessanalyzer list-analyzers
```

**Get findings:**
```bash
# Replace with your actual analyzer ARN from previous command
aws accessanalyzer list-findings \
  --analyzer-arn arn:aws:access-analyzer:eu-central-1:123456789012:analyzer/MyFreeAnalyzer
```

**Expected Output:**
```json
{
  "findings": [
    {
      "id": "12345678-1234-1234-1234-123456789012",
      "resourceType": "AWS::S3::Bucket",
      "resource": "arn:aws:s3:::my-public-demo-bucket-demo123",
      "isPublic": true,
      "status": "ACTIVE",
      "createdAt": "2025-10-22T10:30:00.000Z",
      "analyzedAt": "2025-10-22T10:30:00.000Z",
      "updatedAt": "2025-10-22T10:30:00.000Z"
    }
  ]
}
```

🎉 **Success!** Access Analyzer has detected the public bucket automatically.

---

### Step 6: Remediate and Clean Up

Now let's fix the security issue and clean up our resources.

**1. Remove the public policy:**
```bash
aws s3api delete-bucket-policy \
  --bucket my-public-demo-bucket-demo123
```

**2. Re-enable Block Public Access:**
```bash
aws s3api put-public-access-block \
  --bucket my-public-demo-bucket-demo123 \
  --public-access-block-configuration \
    "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

**3. Delete the bucket:**
```bash
aws s3 rb s3://my-public-demo-bucket-demo123 --force
```

**4. Delete the analyzer:**
```bash
aws accessanalyzer delete-analyzer \
  --analyzer-name MyFreeAnalyzer
```

**Verify cleanup:**
```bash
aws accessanalyzer list-analyzers
# Should return empty list
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     AWS Account                          │
│                                                          │
│  ┌──────────────────────┐                               │
│  │  IAM Access Analyzer │                               │
│  │   (MyFreeAnalyzer)   │                               │
│  └──────────┬───────────┘                               │
│             │ Continuous                                 │
│             │ Monitoring                                 │
│             ▼                                            │
│  ┌──────────────────────┐                               │
│  │     S3 Bucket        │                               │
│  │  (Public Policy)     │◄──────── Bucket Policy        │
│  └──────────┬───────────┘          (Principal: "*")     │
│             │                                            │
│             │ Generates                                  │
│             ▼                                            │
│  ┌──────────────────────┐                               │
│  │      Finding         │                               │
│  │  - Type: Public      │                               │
│  │  - Status: ACTIVE    │                               │
│  │  - Resource: S3      │                               │
│  └──────────────────────┘                               │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 💰 Cost Breakdown

| Service | Usage | Cost |
|---------|-------|------|
| IAM Access Analyzer | Analyzer creation & findings | **$0.00** |
| S3 Bucket | Metadata only (no data stored) | **$0.00** |
| AWS CLI | Local execution | **$0.00** |
| **Total** | | **$0.00** |

### Why It's Free

- IAM Access Analyzer has **no charges** for the core service
- Findings are unlimited and free
- S3 buckets without data incur no storage costs
- API calls within Free Tier limits

---

## 🎯 Best Practices

### 1. Enable Organization-Wide Monitoring

If you manage multiple AWS accounts:

```bash
aws accessanalyzer create-analyzer \
  --analyzer-name OrgAnalyzer \
  --type ORGANIZATION \
  --region us-east-1
```

### 2. Integrate with CI/CD Pipelines

Validate policies before deployment:

```bash
# Add to your CI/CD pipeline
aws accessanalyzer validate-policy \
  --policy-document file://policy.json \
  --policy-type RESOURCE_POLICY

if [ $? -ne 0 ]; then
  echo "❌ Policy validation failed!"
  exit 1
fi
```

### 3. Set Up Automated Alerts

Create an EventBridge rule to notify your team:

```json
{
  "source": ["aws.access-analyzer"],
  "detail-type": ["Access Analyzer Finding"],
  "detail": {
    "status": ["ACTIVE"]
  }
}
```

### 4. Regular Finding Reviews

- **Weekly:** Review new findings
- **Monthly:** Audit archived findings
- **Quarterly:** Validate that expected external access is still necessary

### 5. Archive Expected Findings

For legitimate external access:

```bash
aws accessanalyzer update-findings \
  --analyzer-arn <your-analyzer-arn> \
  --ids <finding-id> \
  --status ARCHIVED
```

### 6. Use Proactive Validation

Always validate policies in development:

```bash
aws accessanalyzer validate-policy \
  --policy-document file://bucket-policy.json \
  --policy-type RESOURCE_POLICY
```

---

## 🔍 Key Concepts

### Analyzer Types

| Type | Scope | Use Case |
|------|-------|----------|
| **Account** | Single AWS account | Individual account monitoring |
| **Organization** | All accounts in AWS Org | Centralized multi-account monitoring |

### Finding Status

- **ACTIVE** - Unresolved external access detected
- **ARCHIVED** - Manually marked as reviewed/expected
- **RESOLVED** - Access automatically removed

### Supported Resources

- Amazon S3 buckets
- IAM roles
- AWS KMS keys
- AWS Lambda functions
- Amazon SQS queues
- Amazon SNS topics
- AWS Secrets Manager secrets
- Amazon ECR repositories
- Amazon EFS file systems

---

## 🐛 Troubleshooting

### Common Issues

**Problem:** `An error occurred (InvalidParameterException) when calling the CreateAnalyzer operation`

**Solution:** Ensure you're using a supported region and have proper IAM permissions.

---

**Problem:** No findings appear after 2 minutes

**Solution:** 
```bash
# Verify the analyzer is active
aws accessanalyzer get-analyzer \
  --analyzer-name MyFreeAnalyzer

# Check the bucket policy was applied
aws s3api get-bucket-policy \
  --bucket my-public-demo-bucket-demo123
```

---

**Problem:** `AccessDenied` errors

**Solution:** Verify your IAM user/role has the required permissions:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "access-analyzer:*",
        "s3:*"
      ],
      "Resource": "*"
    }
  ]
}
```

---

**Problem:** Bucket name already exists

**Solution:** S3 bucket names must be globally unique. Try adding a random suffix:
```bash
aws s3api create-bucket \
  --bucket my-public-demo-bucket-$(date +%s) \
  --region eu-central-1
```

---

## 📚 Additional Resources

### Official AWS Documentation
- [IAM Access Analyzer User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)
- [S3 Bucket Policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html)
- [AWS CLI Reference](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/accessanalyzer/index.html)

### Related Projects
- [AWS Security Hub](https://aws.amazon.com/security-hub/)
- [CloudFormation Guard](https://github.com/aws-cloudformation/cloudformation-guard)
- [Prowler](https://github.com/prowler-cloud/prowler)

### Learning Resources
- [AWS Security Best Practices](https://aws.amazon.com/architecture/security-identity-compliance/)
- [IAM Policy Simulator](https://policysim.aws.amazon.com/)
- [AWS Well-Architected Framework - Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)

---

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

1. **Fork the repository**
2. **Create a feature branch:** `git checkout -b feature/amazing-feature`
3. **Commit your changes:** `git commit -m 'Add amazing feature'`
4. **Push to the branch:** `git push origin feature/amazing-feature`
5. **Open a Pull Request**

### Contribution Ideas

- Additional AWS service examples
- Terraform/CloudFormation templates
- Integration with other security tools
- Troubleshooting guides
- Translations

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 👨‍💻 Author

**Afaq Ul Haq Babar**  
Cloud Engineer & Product Owner - *Open Telekom Cloud / AWS Compute Services*

- 🌐 GitHub: [@afaqbabar](https://github.com/afaqbabar)
- ✍️ Medium: [Your Medium Profile](https://medium.com/@afaqbabar)
- 💼 LinkedIn: [Your LinkedIn](https://linkedin.com/in/afaqbabar)

---

## ⭐ Show Your Support

If you found this guide helpful, please consider:

- Giving it a ⭐ on GitHub
- Sharing it with your network
- Following for more AWS security content

---

## 📝 Changelog

### Version 1.0.0 (October 2025)
- Initial release
- Complete hands-on lab guide
- CLI and Console instructions
- Troubleshooting section

---

## 🙏 Acknowledgments

- AWS Security team for creating Access Analyzer
- The AWS community for continuous feedback
- All contributors who help improve this guide

---

<div align="center">

**[⬆ Back to Top](#️-aws-iam-access-analyzer---hands-on-lab)**

Made with ❤️ for the AWS community

</div>