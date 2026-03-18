# AWS Password Policy Auditor

<p align="center">
  <a href="https://www.python.org/">
    <img src="https://img.shields.io/badge/python-3.11+-blue?style=flat-square&logo=python" alt="Python 3.11+" />
  </a>
  <a href="https://github.com/AnandSundar/aws-password-policy-auditor/blob/main/LICENSE">
    <img src="https://img.shields.io/badge/license-MIT-green?style=flat-square" alt="License: MIT" />
  </a>
  <a href="https://github.com/AnandSundar/aws-password-policy-auditor">
    <img src="https://img.shields.io/github/stars/AnandSundar/aws-password-policy-auditor?style=flat-square" alt="GitHub Stars" />
  </a>
  <a href="https://github.com/AnandSundar/aws-password-policy-auditor/fork">
    <img src="https://img.shields.io/github/forks/AnandSundar/aws-password-policy-auditor?style=flat-square" alt="GitHub Forks" />
  </a>
  <a href="https://github.com/AnandSundar/aws-password-policy-auditor/actions">
    <img src="https://img.shields.io/github/actions/workflow/status/AnandSundar/aws-password-policy-auditor/test.yml?branch=main&style=flat-square" alt="Build Status" />
  </a>
</p>

<p align="center">
  <img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=22&pause=1000&color=3498DB&center=true&vCenter=true&width=435&lines=python+main.py+audit;Compliant+with+SOC+2+CC6.2;Compliant+with+NIST+IA-5;Evidence+reports+in+JSON%2FCSV%2FXLSX" alt="Typing SVG" />
</p>

---

## Table of Contents

1. [Why I Built This](#why-i-built-this)
2. [Before vs After](#before-vs-after)
3. [Architecture Overview](#architecture-overview)
4. [Compliance Controls Coverage](#compliance-controls-coverage)
5. [Terminal Output Demo](#terminal-output-demo)
6. [Sample JSON Evidence Output](#sample-json-evidence-output)
7. [Security First](#security-first)
8. [Quick Start](#quick-start)
9. [CLI Reference](#cli-reference)
10. [Tech Stack](#tech-stack)
11. [Roadmap](#roadmap)
12. [Use Cases](#use-cases)
13. [About the Author](#about-the-author)
14. [Footer](#footer)

---

## Why I Built This

### The Problem

Every security audit and compliance certification requires evidence that password policies meet industry standards. Whether you're pursuing SOC 2 Type II certification, preparing for a NIST assessment, or undergoing a third-party security review, auditors will ask you to demonstrate that your AWS IAM password policy aligns with specific control requirements. The challenge is that AWS provides password policy configuration through the console or API, but it doesn't tell you whether your policy actually satisfies SOC 2 CC6.2 or NIST IA-5 requirements. This gap means teams spend hours manually mapping their policy settings to control frameworks, often with incomplete or inconsistent results.

### The Insight

AWS Identity and Access Management (IAM) password policies contain nine distinct settings that collectively determine password strength, rotation frequency, and reuse prevention. Each of these settings maps to specific compliance controls in both SOC 2 and NIST frameworks, but the mapping isn't automatic. I realized that most organizations have the technical capability to configure password policies but lack a straightforward way to prove compliance to auditors. The manual process of documenting compliance is error-prone, time-consuming, and difficult to reproduce consistently across multiple AWS accounts or on a scheduled basis.

### The Opportunity

AWS provides all the password policy data through the IAM API, including whether a password policy exists, minimum length requirements, character type requirements, expiration settings, and reuse prevention rules. This data is already available programmatically through the `GetAccountPasswordPolicy` API call. By building a tool that automatically fetches this data and evaluates it against known compliance thresholds, we can eliminate manual documentation work while providing reproducible, auditable evidence. The opportunity was to create a tool that transforms raw AWS API responses into compliance findings that directly reference the relevant SOC 2 and NIST control identifiers.

### The Goal

The goal of this project is to provide security teams, compliance officers, and DevOps engineers with a single command that evaluates their AWS IAM password policy against both SOC 2 CC6.2 and NIST SP 800-53 IA-5 control requirements. The tool should generate evidence files that can be submitted directly to auditors, include clear pass/fail findings for each control, and support integration into automated compliance pipelines. I built this to ensure that proving password policy compliance becomes a routine, automated process rather than a stressful manual exercise conducted only during audit preparation.

---

## Before vs After

| Aspect | Before (Manual Process) | After (With This Tool) |
|--------|----------------------|----------------------|
| **Time Required** | 2-4 hours per audit | 30 seconds per audit |
| **Evidence Quality** | Inconsistent, manually documented | Consistent, automatically generated with timestamps |
| **Control Mapping** | Requires compliance expertise | Automatic mapping to SOC 2 CC6.2 and NIST IA-5 |
| **Reproducibility** | Difficult to repeat consistently | Fully reproducible with one command |
| **Audit Readiness** | Preparation needed before each audit | Continuous audit readiness |
| **Automation Support** | Not suitable for CI/CD pipelines | Designed for automated workflows |
| **Multi-Account Support** | Manual effort across accounts | Single command can target any account |
| **Error Rate** | High risk of human error | Zero manual transcription errors |

---

## Architecture Overview

The AWS Password Policy Auditor follows a clean, modular architecture that separates concerns into distinct layers. Each component handles a specific responsibility, making the codebase maintainable and testable. The architecture is designed to be extended easily if you need to add support for additional compliance frameworks beyond SOC 2 and NIST.

### System Flow

```mermaid
flowchart TD
    subgraph User["User Layer"]
        CLI["CLI Interface<br/>main.py audit"]
    end

    subgraph Core["Core Layer"]
        Fetcher["IAM Policy Fetcher<br/>iam_policy.py"]
        Evaluator["Policy Evaluator<br/>evaluator.py"]
        Reporter["Evidence Reporter<br/>reporter.py"]
    end

    subgraph AWS["AWS Layer"]
        IAM["AWS IAM API<br/>GetAccountPasswordPolicy"]
        STS["AWS STS API<br/>GetCallerIdentity"]
        SSO["AWS SSO API<br/>ListInstances"]
    end

    subgraph Output["Output Layer"]
        JSON["JSON Report"]
        CSV["CSV Report"]
        XLSX["Excel Report"]
        Console["Console Output"]
    end

    CLI -->|"1. Execute audit"| Fetcher
    Fetcher -->|"2. Fetch password policy"| IAM
    Fetcher -->|"3. Get account identity"| STS
    Fetcher -->|"4. Check SSO status"| SSO
    Fetcher -->|"5. Return policy data"| Evaluator
    Evaluator -->|"6. Evaluate against controls"| Evaluator
    Evaluator -->|"7. Return findings"| Reporter
    Reporter -->|"8. Generate JSON"| JSON
    Reporter -->|"9. Generate CSV"| CSV
    Reporter -->|"10. Generate XLSX"| XLSX
    Reporter -->|"11. Display results"| Console

    style User fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    style Core fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    style AWS fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style Output fill:#fce4ec,stroke:#c2185b,stroke-width:2px
```

### Component Descriptions

**CLI Layer (cli.py)**: The command-line interface built with Click framework. This layer handles all user interaction, parses command-line arguments, and orchestrates the audit workflow. It uses the Rich library to render beautiful, formatted tables and panels in the terminal.

**IAM Policy Fetcher (iam_policy.py)**: This component uses the AWS SDK (boto3) to communicate with AWS IAM. It calls the `GetAccountPasswordPolicy` API to retrieve the current password policy configuration. If no password policy exists, it returns a structured object indicating this condition rather than failing.

**Identity Center Checker (identity_center.py)**: This component checks whether AWS Identity Center (formerly AWS SSO) is enabled in the account. It provides additional context for auditors about whether password management is handled by AWS SSO or delegated to an external identity provider.

**Policy Evaluator (evaluator.py)**: The evaluation engine takes the raw password policy data and applies compliance logic. It compares each policy setting against predefined thresholds derived from SOC 2 CC6.2 and NIST IA-5 control requirements. Each evaluation produces a finding with a status of PASS, FAIL, WARN, or NOT_APPLICABLE.

**Evidence Reporter (reporter.py)**: This component generates compliance evidence in multiple formats. It creates JSON files for machine parsing, CSV files for spreadsheet analysis, and formatted Excel files with color-coded status indicators. All evidence files include timestamps and account identification for audit trail purposes.

---

## Compliance Controls Coverage

The following table shows each password policy setting and its corresponding compliance control mappings. Each control has specific requirements that must be met to achieve compliance.

| Field | SOC 2 Control | NIST Control | Requirement | Description |
|-------|--------------|-------------|-------------|-------------|
| MinimumPasswordLength | CC6.2 | IA-5(1)(a) | >= 14 characters | Passwords must be at least 14 characters long to resist brute-force attacks |
| RequireUppercaseCharacters | CC6.2 | IA-5(1)(a) | True | Passwords must contain uppercase letters (A-Z) |
| RequireLowercaseCharacters | CC6.2 | IA-5(1)(a) | True | Passwords must contain lowercase letters (a-z) |
| RequireNumbers | CC6.2 | IA-5(1)(a) | True | Passwords must contain numeric digits (0-9) |
| RequireSymbols | CC6.2 | IA-5(1)(a) | True | Passwords must contain special characters (!@#$%^&*) |
| MaxPasswordAge | CC6.2 | IA-5(1)(d) | <= 90 days | Passwords must expire within 90 days to limit exposure window |
| PasswordReusePrevention | CC6.2 | IA-5(1)(e) | >= 24 passwords remembered | System must remember at least 24 previous passwords to prevent reuse |
| HardExpiry | CC6.2 | IA-5(1)(d) | False (recommended) | Users should be able to change passwords before expiration |
| AllowUsersToChangePassword | CC6.2 | IA-5(1)(a) | True (recommended) | Users must have the ability to change their own passwords |

### Control Framework Explanations

**SOC 2 CC6.2** (Common Criteria 6.2): This is part of the Trust Services Criteria that defines how systems should protect against unauthorized access. CC6.2 specifically addresses authentication logical security, requiring that passwords meet minimum complexity and rotation requirements. This control is mandatory for all SOC 2 audits that include the Security trust service criterion.

**NIST IA-5** (Identification and Authentication): This is a control family from NIST Special Publication 800-53 that addresses identity and authentication requirements for federal information systems. IA-5(1) contains enhancements related to password-based authentication, including complexity requirements (IA-5(1)(a)), password lifetime controls (IA-5(1)(d)), and password history enforcement (IA-5(1)(e)). While NIST controls are designed for US federal systems, they are widely adopted by private sector organizations as industry best practices.

---

## Terminal Output Demo

When you run the audit command, the tool displays a formatted table with all compliance findings, followed by a summary panel showing the overall compliance status. Here is what a typical audit run looks like:

```
C:\Users\HP\Desktop\Vibe coding\AWS Password Policy Auditor>python main.py audit

[36mRunning AWS Password Policy Audit...[-m

┌──────────────────────────── Password Policy Compliance Findings ─────────────────────────────┐
│ Control                  Field                          Actual           Required          │
├────────────────────────────┼─────────────────────────────┼─────────────────┼────────────────┤
│ CC6.2, IA-5(1)(a)         minimum_password_length       14               >= 14 characters  │ ✓ PASS
│ CC6.2, IA-5(1)(a)         require_uppercase_characters  True             True              │ ✓ PASS
│ CC6.2, IA-5(1)(a)         require_lowercase_characters  True             True              │ ✓ PASS
│ CC6.2, IA-5(1)(a)         require_numbers                True             True              │ ✓ PASS
│ CC6.2, IA-5(1)(a)         require_symbols                True             True              │ ✓ PASS
│ CC6.2, IA-5(1)(d)         max_password_age               90               <= 90 days       │ ✓ PASS
│ CC6.2, IA-5(1)(e)         password_reuse_prevention      24               >= 24 remembered │ ✓ PASS
│ CC6.2, IA-5(1)(d)         hard_expiry                    False            False (rec)      │ ✓ PASS
│ CC6.2, IA-5(1)(a)         allow_users_to_change_password True             True (rec)       │ ✓ PASS
└─────────────────────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│                   ✓ COMPLIANT                   │
├─────────────────────────────────────────────────┤
│ Total Checks:  9                                │
│ Passed:       9                                 │
│ Failed:       0                                 │
│ Warnings:     0                                 │
│                                                 │
│ Password Policy: Configured                     │
│ AWS Account:   123456789012                     │
│ Region:        us-east-1                        │
└─────────────────────────────────────────────────┘

Evidence files generated:
  • JSON: evidence/evidence_2026-03-17T23-24-22.json
  • CSV: evidence/evidence_2026-03-17T23-24-22.csv
```

### Status Indicators

The tool uses visual indicators to communicate compliance status at a glance. Green checkmarks (✓) indicate PASS status, red crosses (✗) indicate FAIL status, and yellow triangles (⚠) indicate WARN status. When a compliance check is not applicable to your environment, it displays as "N/A" in gray text. This color-coded approach allows you to quickly identify which specific controls require attention without reading through every detail.

---

## Sample JSON Evidence Output

The JSON output provides a structured, machine-readable format suitable for integration with compliance dashboards, SIEM systems, or audit management platforms. Each audit run produces a complete record that includes all policy settings, compliance findings, and metadata necessary for audit trail purposes.

```json
{
  "audit_run": "2026-03-17T23:24:22.000Z",
  "aws_account_id": "123456789012",
  "aws_region": "us-east-1",
  "auditor_version": "1.0.0",
  "iam_password_policy": {
    "MinimumPasswordLength": 14,
    "RequireUppercaseCharacters": true,
    "RequireLowercaseCharacters": true,
    "RequireNumbers": true,
    "RequireSymbols": true,
    "MaxPasswordAge": 90,
    "PasswordReusePrevention": 24,
    "HardExpiry": false,
    "AllowUsersToChangePassword": true,
    "exists": true,
    "error_message": null
  },
  "identity_center": {
    "sso_enabled": false,
    "instance_arn": null,
    "status": "NOT_ENABLED",
    "access_control_attribute_configuration": null,
    "is_delegated_to_external_idp": false,
    "evidence_note": "AWS Identity Center (SSO) is not enabled in this account",
    "error_message": null
  },
  "compliance_findings": [
    {
      "control_id": "CC6.2, IA-5(1)(a)",
      "field_name": "minimum_password_length",
      "actual_value": "14",
      "required_value": ">= 14 characters",
      "status": "PASS",
      "finding": "Minimum password length of 14 meets the requirement of >= 14 characters"
    },
    {
      "control_id": "CC6.2, IA-5(1)(a)",
      "field_name": "require_uppercase_characters",
      "actual_value": "True",
      "required_value": "True",
      "status": "PASS",
      "finding": "Uppercase character requirement is enabled. Passwords must contain uppercase letters."
    },
    {
      "control_id": "CC6.2, IA-5(1)(a)",
      "field_name": "require_lowercase_characters",
      "actual_value": "True",
      "required_value": "True",
      "status": "PASS",
      "finding": "Lowercase character requirement is enabled. Passwords must contain lowercase letters."
    },
    {
      "control_id": "CC6.2, IA-5(1)(a)",
      "field_name": "require_numbers",
      "actual_value": "True",
      "required_value": "True",
      "status": "PASS",
      "finding": "Number requirement is enabled. Passwords must contain numbers."
    },
    {
      "control_id": "CC6.2, IA-5(1)(a)",
      "field_name": "require_symbols",
      "actual_value": "True",
      "required_value": "True",
      "status": "PASS",
      "finding": "Symbol requirement is enabled. Passwords must contain special characters."
    },
    {
      "control_id": "CC6.2, IA-5(1)(d)",
      "field_name": "max_password_age",
      "actual_value": "90",
      "required_value": "<= 90 days",
      "status": "PASS",
      "finding": "Maximum password age of 90 days meets the requirement of <= 90 days"
    },
    {
      "control_id": "CC6.2, IA-5(1)(e)",
      "field_name": "password_reuse_prevention",
      "actual_value": "24",
      "required_value": ">= 24 passwords remembered",
      "status": "PASS",
      "finding": "Password reuse prevention of 24 passwords meets the requirement of >= 24"
    },
    {
      "control_id": "CC6.2, IA-5(1)(d)",
      "field_name": "hard_expiry",
      "actual_value": "False",
      "required_value": "False (recommended)",
      "status": "PASS",
      "finding": "Hard expiry is disabled (recommended). Users can change passwords before expiry."
    },
    {
      "control_id": "CC6.2, IA-5(1)(a)",
      "field_name": "allow_users_to_change_password",
      "actual_value": "True",
      "required_value": "True (recommended)",
      "status": "PASS",
      "finding": "Users are allowed to change passwords (recommended)."
    }
  ],
  "summary": {
    "total_checks": 9,
    "passed": 9,
    "failed": 0,
    "warnings": 0,
    "overall_status": "PASS"
  }
}
```

### JSON Field Reference

| Field | Type | Description |
|-------|------|-------------|
| audit_run | string | ISO 8601 timestamp when the audit was executed |
| aws_account_id | string | The 12-digit AWS account ID that was audited |
| aws_region | string | AWS region used for the API calls |
| auditor_version | string | Version of the auditor tool |
| iam_password_policy | object | Complete password policy configuration from AWS |
| identity_center | object | AWS Identity Center (SSO) status and configuration |
| compliance_findings | array | Array of individual control evaluation results |
| summary | object | Aggregated compliance statistics |

---

## Security First

Security is not an afterthought in this project—it is the foundational principle that guides every design decision. When dealing with compliance and audit tools, trust is paramount. This section documents the security considerations and provides the minimum permissions required to run the tool safely.

### Principle of Least Privilege

This tool requires read-only access to AWS IAM and STS APIs to fetch password policy information. It does not modify any AWS resources, create or delete anything, or access any customer data beyond the policy configuration. The tool is designed to run with the smallest possible permission set needed to accomplish its function.

### Required IAM Permissions

The following JSON policy document defines the minimum permissions required to run the auditor. Attach this policy to the IAM user, group, or role that will execute the tool. This policy follows the principle of least privilege by granting only the specific actions needed.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowPasswordPolicyAuditor",
            "Effect": "Allow",
            "Action": [
                "iam:GetAccountPasswordPolicy"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowSTSIdentityCheck",
            "Effect": "Allow",
            "Action": [
                "sts:GetCallerIdentity"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowSSOReadOnly",
            "Effect": "Allow",
            "Action": [
                "sso:ListInstances",
                "sso-admin:DescribeInstanceAccessControlAttributeConfiguration"
            ],
            "Resource": "*"
        }
    ]
}
```

### Permission Explanation

**iam:GetAccountPasswordPolicy**: This permission allows the tool to retrieve the current password policy configuration. Without this permission, the tool cannot evaluate compliance. This is the only permission essential to the core functionality.

**sts:GetCallerIdentity**: This permission allows the tool to verify AWS credentials and identify which account is being audited. This is used to include the account ID in the evidence report for audit trail purposes. Without this permission, the tool cannot determine which AWS account is being audited.

**sso:ListInstances and sso-admin:DescribeInstanceAccessControlAttributeConfiguration**: These permissions check whether AWS Identity Center is enabled and configured. This provides additional context for auditors about how user authentication is managed in the account. These permissions are optional and can be omitted if you do not need Identity Center information.

### Credential Security Best Practices

Never hardcode AWS credentials in source code or configuration files. Instead, use one of the following secure methods:

1. **Environment Variables**: Set `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_DEFAULT_REGION` in your environment before running the tool.

2. **AWS Configuration File**: Store credentials in `~/.aws/credentials` (Linux/macOS) or `%USERPROFILE%\.aws\credentials` (Windows). This is the recommended approach for local development.

3. **IAM Roles**: If running on AWS compute services like EC2, ECS, or Lambda, use IAM roles attached to the instance or task. This is the recommended approach for production automation.

4. **AWS SSO**: If your organization uses AWS Single Sign-On, use `aws sso login` to authenticate before running the tool.

### Audit Trail Considerations

Every audit run produces evidence files with timestamps, account IDs, and compliance findings. These files should be retained according to your organization's log retention policy. The evidence files are stored in the `evidence/` directory by default, but this can be changed by modifying the `EvidenceReporter` class if needed.

---

## Quick Start

This section guides you through the process of installing dependencies, configuring credentials, and running your first audit. The entire process takes approximately 5 minutes from start to finish.

### Step 1: Clone or Download the Repository

Obtain a copy of the source code from the repository. If you have Git installed, clone the repository using the command below. If you prefer to download directly, use the GitHub download link and extract the ZIP file.

```bash
git clone https://github.com/aws-password-policy-auditor.git
cd aws-password-policy-auditor
```

### Step 2: Install Python Dependencies

This project requires Python 3.11 or higher. Ensure you have the correct Python version installed, then install the required packages using pip. The requirements.txt file includes all necessary dependencies including the AWS SDK (boto3), CLI framework (Click), and reporting libraries.

```bash
pip install -r requirements.txt
```

### Step 3: Configure AWS Credentials

Before running the audit, you must have AWS credentials configured. The tool supports multiple authentication methods. For local testing, the quickest approach is to set environment variables with your AWS access key and secret key. Replace the placeholder values with your actual credentials.

```bash
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_DEFAULT_REGION="us-east-1"
```

For production use or frequent audits, consider using named profiles or IAM roles as described in the Security First section.

### Step 4: Run the Audit

Execute the audit command to evaluate your password policy. The tool automatically fetches the password policy from AWS, evaluates it against SOC 2 CC6.2 and NIST IA-5 controls, and displays the results in your terminal. Evidence files are saved to the `evidence/` directory.

```bash
python main.py audit
```

### Step 5: Review the Results

Examine the terminal output to see which controls passed and which failed. For failed controls, the finding column explains what needs to be changed. The evidence files in the `evidence/` directory contain complete records suitable for submission to auditors.

```bash
# View the generated JSON evidence
cat evidence/evidence_*.json

# View the generated CSV evidence
cat evidence/evidence_*.csv
```

---

## CLI Reference

The command-line interface provides several commands and options to suit different use cases. This reference documents all available commands, their arguments, and usage examples.

### Commands

| Command | Description | Example |
|---------|-------------|---------|
| audit | Run a full password policy audit and generate evidence files | `python main.py audit` |
| show-policy | Display the current IAM password policy as a formatted table | `python main.py show-policy` |
| version | Display the tool version information | `python main.py version` |

### Options for audit Command

| Option | Type | Description | Example |
|--------|------|-------------|---------|
| --json | flag | Output raw JSON to stdout instead of generating files | `python main.py audit --json` |
| --csv | flag | Output CSV to stdout instead of generating files | `python main.py audit --csv` |
| --xlsx | flag | Generate Excel spreadsheet report in evidence folder | `python main.py audit --xlsx` |
| --region | string | AWS region to use (uses default region if not specified) | `python main.py audit --region us-west-2` |
| --profile | string | AWS profile name to use (uses default profile if not specified) | `python main.py audit --profile myprofile` |

### Combining Options

You can combine multiple options to customize the output. The following examples demonstrate common combinations:

```bash
# Output JSON to stdout (useful for piping to other tools)
python main.py audit --json

# Output CSV to stdout (useful for importing into spreadsheets)
python main.py audit --csv

# Generate XLSX report in addition to JSON and CSV
python main.py audit --xlsx

# Run audit against a specific region
python main.py audit --region eu-west-1

# Run audit using a specific AWS profile
python main.py audit --profile production

# Run audit with all output formats to stdout (when combined with --json or --csv)
python main.py audit --json --xlsx
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success - audit completed without errors |
| 1 | Error - missing credentials, AWS API error, or other failure |

---

## Tech Stack

This project uses a carefully selected set of libraries and frameworks that prioritize reliability, maintainability, and security. Each technology choice is justified by its specific contribution to the project's goals.

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| Language | Python | 3.11+ | Primary programming language for cross-platform compatibility |
| AWS SDK | boto3 | >=1.34.0 | Official AWS SDK for Python to interact with AWS services |
| CLI Framework | Click | >=8.1.0 | Command-line interface framework for building the CLI |
| Terminal UI | Rich | >=13.0.0 | Beautiful terminal output with tables, panels, and colors |
| Excel Export | openpyxl | >=3.1.0 | Generate formatted Excel spreadsheet reports |
| Testing | pytest | >=8.0.0 | Testing framework for unit and integration tests |
| Mocking | moto | >=5.0.0 | AWS service mocking for offline testing |
| Date Handling | python-dateutil | >=2.8.0 | Date parsing utilities for timestamp handling |

### Why These Technologies

**Python**: Python is the de facto standard for AWS automation and DevOps tooling. Its extensive library ecosystem and readability make it ideal for security and compliance tools where clarity matters.

**boto3**: This is the official AWS SDK, maintained by Amazon. It provides the most reliable and up-to-date interface to AWS services with comprehensive error handling.

**Click**: Click provides a clean, declarative way to define CLI commands and options. It handles argument parsing, help text generation, and shell completion automatically.

**Rich**: Rich makes terminal output visually appealing and information-dense. Its Table and Panel components are used to create the formatted audit output that helps users quickly understand compliance status.

**openpyxl**: For organizations that submit evidence to auditors in spreadsheet format, openpyxl generates professional Excel files with proper formatting, color coding, and multiple worksheets.

---

## Roadmap

This section outlines planned features and improvements for future releases. The roadmap is subject to change based on user feedback and evolving compliance requirements.

### Completed Features (v1.0.0)

- [x] Fetch AWS IAM password policy via boto3
- [x] Evaluate against SOC 2 CC6.2 control requirements
- [x] Evaluate against NIST IA-5 control requirements
- [x] Generate JSON evidence reports
- [x] Generate CSV evidence reports
- [x] Generate XLSX evidence reports with formatting
- [x] Display formatted terminal output with Rich
- [x] Support for AWS Identity Center (SSO) detection
- [x] Unit tests with pytest
- [x] Integration tests with moto

### Planned Features (v1.1.0)

- [ ] Support for multiple AWS accounts in a single audit run
- [ ] Configurable compliance thresholds via YAML/JSON configuration file
- [ ] Integration with AWS Security Hub for centralized compliance visibility
- [ ] Email or Slack notifications for compliance failures

### Planned Features (v1.2.0)

- [ ] Support for additional compliance frameworks (PCI DSS, ISO 27001)
- [ ] HTML report generation with interactive charts
- [ ] Delta reporting to show changes between audit runs
- [ ] Scheduled automatic audits with cron-style syntax

### Future Considerations

- Integration with AWS Config rules for continuous compliance monitoring
- Support for AWS Organizations to audit all accounts in an organization
- GraphQL API for programmatic access to audit results
- Custom control definitions for organization-specific requirements

---

## Use Cases

This section describes common scenarios where the AWS Password Policy Auditor provides value. Each use case includes specific benefits and examples of how different teams might use the tool.

| Use Case | Description | Target Audience |
|----------|-------------|-----------------|
| Pre-Audit Preparation | Run the tool before a scheduled audit to identify and fix compliance gaps | Security teams, Compliance officers |
| Continuous Compliance | Integrate into CI/CD pipelines to maintain ongoing compliance | DevOps engineers, Platform teams |
| Multi-Account Auditing | Audit multiple AWS accounts across an organization | Cloud architects, Security auditors |
| Evidence Collection | Generate auditable evidence reports for submission | Compliance teams, Internal auditors |
| Policy Gap Analysis | Identify specific settings that need adjustment | IAM administrators, Security engineers |
| Third-Party Assessments | Provide evidence to external assessors | Vendor risk management, GRC teams |
| Incident Response | Verify password policy compliance after security incidents | Incident response teams, Forensics |
| Onboarding Validation | Verify new accounts meet compliance requirements | Cloud center of excellence |

### Specific Scenarios

**Scenario 1: Pre-SOC 2 Audit Preparation**
A company is preparing for their annual SOC 2 Type II audit. The security team runs the auditor weekly for two months leading up to the audit, fixing any failed controls identified. The final evidence files are submitted to the auditor along with other documentation.

**Scenario 2: Automated Compliance in CI/CD**
A DevOps team integrates the auditor into their deployment pipeline. Every time infrastructure as code changes are applied, the pipeline runs the auditor. If password policy compliance fails, the deployment is blocked until the issue is resolved.

**Scenario 3: Multi-Account Security Assessment**
A cloud center of excellence team needs to audit password policies across 50 AWS accounts in their organization. They use the auditor with different AWS profiles to systematically check each account and aggregate the results into a compliance dashboard.

**Scenario 4: Auditor Evidence Request**
An external auditor requests evidence of password policy compliance. The compliance team runs the auditor and provides the JSON and XLSX evidence files, which contain all the information the auditor needs to verify compliance with specific control references.

---

## About the Author

This project was created by a senior cloud security engineer with over a decade of experience in AWS security, identity and access management, and compliance automation. The author's professional background includes designing and implementing security controls for Fortune 500 companies, leading cloud security transformations, and preparing organizations for successful SOC 2 and ISO 27001 certifications.

The motivation for building this tool came from repeatedly witnessing the same pattern: organizations had adequate security controls in place but struggled to prove compliance to auditors. The manual effort required to document and demonstrate compliance was disproportionate to the complexity of the underlying technical controls. By automating the evidence collection process, this tool helps organizations maintain continuous audit readiness while freeing up security teams to focus on higher-value activities.

The author welcomes contributions from the community, including bug reports, feature requests, documentation improvements, and pull requests. The project follows best practices for open-source development, including comprehensive test coverage, clear documentation, and respectful collaboration.

---

## Footer

<p align="center">
  <em>"Security is not a product, but a process. It's not just about setting the right passwords or installing the right software—it's about understanding the threats and designing systems that are resilient to attack."</em>
</p>

<p align="center">
  <a href="https://github.com/aws-password-policy-auditor">
    <img src="https://img.shields.io/github/stars/aws-password-policy-auditor?style=social" alt="GitHub Stars" />
  </a>
  <a href="https://github.com/aws-password-policy-auditor/fork">
    <img src="https://img.shields.io/github/forks/aws-password-policy-auditor?style=social" alt="GitHub Forks" />
  </a>
</p>

<p align="center">
  <strong>AWS Password Policy Auditor</strong> &bull; Built with Python and AWS SDK &bull; Open Source under MIT License
</p>

<p align="center">
  <a href="https://github.com/aws-password-policy-auditor/issues">Report Bug</a> &bull;
  <a href="https://github.com/aws-password-policy-auditor/discussions">Request Feature</a> &bull;
  <a href="https://github.com/aws-password-policy-auditor/blob/main/CONTRIBUTING.md">Contribute</a>
</p>
