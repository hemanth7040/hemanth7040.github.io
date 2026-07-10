---
title: "EC2 Inventory Generator"
date: 2026-07-10
description: "Automating AWS EC2 inventory collection using Python and Boto3."
summary: "A Python-based automation tool that generates EC2 inventory reports from AWS."
projects: ["python"]
draft: false

tags:
  - AWS
  - EC2
  - Python
  - Boto3
  - DevOps
  - Automation

categories:
  - AWS Automation
  - Cloud

tech:
  - Python
  - AWS EC2
  - Boto3
  - CSV
  - Logging

github: "https://github.com/<your-github>/ec2-inventory-generator"

weight: 4
---

## Overview

Managing AWS environments becomes increasingly difficult as the number of EC2 instances grows. While the AWS Management Console provides detailed information, manually collecting instance details for reporting, auditing, or documentation is repetitive and time-consuming.

To simplify this process, I built the **EC2 Inventory Generator**, a Python-based automation tool that connects to AWS using the **Boto3 SDK**, retrieves EC2 instance information, extracts commonly used resource tags, and generates a clean CSV inventory report.

This project demonstrates practical cloud automation using Python and AWS APIs while following a modular and maintainable project structure.

---

## Why I Built This

During infrastructure management, I often needed a quick inventory of EC2 instances for documentation and operational tasks.

Instead of manually navigating through the AWS Console and copying information into spreadsheets, I wanted a repeatable solution that could automatically generate an up-to-date inventory report in seconds.

The project helped me explore:

- AWS SDK (Boto3)
- EC2 APIs
- Python automation
- CSV report generation
- Logging and error handling
- Modular application design

---

## What Problem Does It Solve?

Without automation, cloud teams often spend valuable time gathering information such as:

- Instance IDs
- Instance names
- Running or stopped status
- Public and private IP addresses
- VPC and subnet details
- Resource tags

This process becomes increasingly difficult as infrastructure scales.

The EC2 Inventory Generator automates this task by collecting the information directly from AWS and exporting it into a structured CSV file.

---

## Architecture

```text
              AWS Credentials
                     │
                     ▼
            Python Application
                     │
                     ▼
               Boto3 EC2 Client
                     │
                     ▼
      EC2 DescribeInstances API
                     │
                     ▼
        Process Instance Metadata
                     │
          Extract Resource Tags
                     │
                     ▼
          Generate CSV Inventory
                     │
                     ▼
             Inventory Report
```
---

## Project Structure

```text
ec2-inventory/
│
├── main.py
├── aws_inventory.py
├── logger.py
├── requirements.txt
│
├── output/
│   └── inventory.csv
│
├── logs/
│   └── inventory.log
│
└── README.md
```

The project is organized into separate modules to keep the application easy to understand and maintain.

| Module | Responsibility |
|---|---|
| `main.py` | Starts the application |
| `aws_inventory.py` | Contains the AWS logic |
| `logger.py` | Handles application logging |
| `output/` | Stores generated reports |
| `logs/` | Stores execution logs |

---

## How It Works

The application follows a straightforward workflow:

1. **Authenticate** using AWS credentials configured on the local machine.
2. **Connect** to Amazon EC2 using the Boto3 SDK.
3. **Call** the EC2 `DescribeInstances` API.
4. **Collect** information about every EC2 instance.
5. **Extract** commonly used resource tags.
6. **Generate** a CSV inventory report.
7. **Record** execution details in the log file.

---

## Features

### Automatic EC2 Discovery

The application automatically discovers all EC2 instances in the configured AWS account and region.

### Instance Metadata Collection

For every EC2 instance, the tool collects:

- Instance Name
- Instance ID
- Instance Type
- Current State
- Launch Time
- Availability Zone
- VPC ID
- Subnet ID
- Private IP Address
- Public IP Address

### Resource Tag Extraction

AWS stores metadata as tags. The application automatically extracts tags such as:

- Name
- Owner
- Environment
- Project

If a tag is missing, the application safely skips it instead of generating an error.

### CSV Report Generation

All collected information is exported into a CSV file that can be opened using Excel or imported into reporting tools.

**Example output:**

| Name | Instance ID | State | Type | Private IP |
|------|-------------|-------|------|-------------|
| WebServer | i-123456 | Running | t3.micro | 10.0.1.15 |

> ![alt text](<Screenshot 2026-07-10 at 3.46.36 PM.png>)

### Logging

Every execution generates a log file that records:

- Script start
- Successful execution
- AWS API errors
- Unexpected exceptions

This makes troubleshooting much easier.

> ![alt text](<Screenshot 2026-07-10 at 3.48.35 PM.png>)

---

## Implementation Highlights

### Connecting to AWS

The application uses **Boto3**, AWS's official Python SDK. A connection is established to the EC2 service before requesting instance information.

```python
response = ec2.describe_instances()
```

This API returns information about all EC2 instances in the configured AWS account.

### Processing the Response

AWS returns instance data as nested JSON. The application loops through each reservation and instance to extract only the required fields.

```python
for reservation in response["Reservations"]:
    for instance in reservation["Instances"]:
        print(instance["InstanceId"])
```

### Handling Missing Values

Not every EC2 instance contains:

- Public IP
- Name tag
- Owner tag

Instead of causing the application to fail, optional values are accessed using Python's `.get()` method. This improves reliability and makes the script suitable for real-world environments.

### Modular Design

The project separates responsibilities into different modules. Instead of placing all logic in one file, each module has a single responsibility.

Benefits include:

- Easier maintenance
- Better readability
- Improved code reuse
- Simpler debugging

---

## Running the Project

**1. Clone the repository**

```bash
git clone https://github.com/hemanth7040/Python-Automation.git
```

**2. Change directory**

```bash
cd 01-ec2-inventory-generator
```

**3. Install dependencies**

```bash
pip install -r requirements.txt
```

**4. Configure AWS credentials**

```bash
aws configure
```

**5. Run the application**

```bash
python main.py
```

The generated report will be available inside the **output/** directory.

> ![alt text](<Screenshot 2026-07-10 at 3.49.48 PM.png>)

---

## Skills Demonstrated

This project demonstrates practical experience with:

- Python programming
- AWS EC2
- Boto3 SDK
- Cloud automation
- Infrastructure reporting
- CSV data processing
- Logging
- Error handling
- Modular software design

---

## Challenges & Lessons Learned

While developing this project, I encountered several real-world scenarios that required careful handling.

Some EC2 instances did not have public IP addresses, while others were missing resource tags. Rather than assuming every field would always be present, I implemented safe access patterns using Python's `.get()` method and reusable helper functions for tag extraction.

This experience reinforced the importance of writing automation scripts that can gracefully handle incomplete or inconsistent cloud resource data.

---

## Future Improvements

Possible enhancements include:

- Multi-region inventory collection
- Cross-account inventory
- Excel export
- JSON export
- Upload reports to Amazon S3
- AWS Lambda integration
- EventBridge scheduling
- HTML dashboard
- Email reports
- Tag compliance validation

---

## What I Learned

This project helped strengthen my understanding of:

- Working with AWS APIs
- Using the Boto3 SDK
- Processing nested JSON responses
- Python file handling
- Infrastructure automation
- Logging best practices
- Writing maintainable Python applications

---

## Conclusion

The EC2 Inventory Generator demonstrates how cloud automation can eliminate repetitive operational tasks.

By combining Python with AWS APIs, the application automatically generates a structured EC2 inventory report that can be used for documentation, auditing, and infrastructure management.

Although the project is relatively lightweight, it showcases several important DevOps concepts, including cloud automation, API integration, data processing, and modular software design.