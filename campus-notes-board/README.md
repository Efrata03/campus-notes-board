
# Campus Notes Board

## 1. Overview and Architecture Diagram

Campus Notes Board is a web application deployed on AWS that allows students to upload, store, and download course notes. The application uses a Flask backend, an EC2 instance for hosting, an RDS MySQL database for storing note metadata, and Amazon S3 for storing uploaded files.

### Architecture

```text
                         Internet
                            |
                     HTTP (Port 80)
                            |
                    Internet Gateway
                            |
                     Public Subnet
                            |
                    EC2 App Instance
                 Ubuntu + Nginx + Flask
                            |
              +-------------+-------------+
              |                           |
         Private Network              Amazon S3
              |                    Private Bucket
          RDS MySQL                 Stores PDF Files
              |
       Notes Metadata
```

### AWS Services Used

- Amazon VPC — isolated network for the project.
- Public subnet — hosts the application EC2 instance.
- Private subnets — used for the RDS database subnet group.
- Amazon EC2 — runs the web application.
- Amazon RDS MySQL — stores uploaded notes metadata.
- Amazon S3 — stores uploaded PDF files.
- AWS IAM — provides permissions to the EC2 instance.
- AWS Systems Manager Session Manager — allows secure server management.
- Amazon CloudWatch — monitors EC2 CPU usage and provides alarms.
- AWS CloudTrail — records AWS management events.

---

## 2. Prerequisites

- AWS account
- AWS Builder ID authentication (used instead of the traditional root/IAM-user login model)
- AWS Management Console access
- Git
- Basic knowledge of Linux commands
- Python 3
- MySQL
- Web browser
- GitHub repository

---

## 3. Deployment Steps

### Step 1: Create the Network

- Created a VPC named `campus-notes-vpc`.
- Created a public subnet for the application server.
- Created private subnets for the database.
- Configured route tables and an Internet Gateway.
- Created security groups to control network traffic.

### Step 2: Create the S3 Bucket

- Created a private S3 bucket for uploaded notes.
- Enabled Block Public Access.
- Enabled server-side encryption.
- Used bucket-owner-enforced object ownership.

### Step 3: Configure IAM

- Created an EC2 IAM role named `notes-ec2-role`.
- Attached Systems Manager permissions.
- Created a custom S3 policy for the application.
- Allowed the application to access only the required S3 resources.

### Step 4: Launch EC2

- Launched an Ubuntu EC2 instance.
- Attached the IAM role.
- Installed and configured the application dependencies.
- Configured Nginx as a reverse proxy.
- Configured Gunicorn to run the Flask application.
- Created a systemd service to manage the application.

### Step 5: Create the Database

- Created an Amazon RDS MySQL database.
- Configured the database to remain private.
- Created the database and application user.
- Allowed database traffic only from the application security group.

### Step 6: Deploy the Application

- Cloned the project repository onto the EC2 instance.
- Created a Python virtual environment.
- Installed application dependencies.
- Configured environment variables outside the Git repository.
- Started the application using systemd.

### Step 7: Verify the Application

- Opened the application through the EC2 public IP.
- Tested uploading a PDF file.
- Verified the note metadata in MySQL.
- Verified that the file was stored in S3.
- Tested downloading the uploaded file.
- Verified the application health endpoint.

---

## 4. Configuration

The application uses environment variables for configuration.

| Variable | Purpose |
|---|---|
| `DB_HOST` | Database hostname |
| `DB_PORT` | Database port |
| `DB_NAME` | Database name |
| `DB_USER` | Database username |
| `DB_PASSWORD` | Database password |
| `S3_BUCKET` | S3 bucket name |
| `AWS_REGION` | AWS region |
| `MAX_UPLOAD_MB` | Maximum upload size |
| `DB_SSL_CA` | Optional database SSL certificate path |

Sensitive configuration is stored in `/etc/notes/notes.env` on the EC2 instance, outside the Git repository. The file is owned by the application user and has restricted permissions.

No passwords or secret values are included in this README.

---

## 5. Database and S3 Setup

### Database

Amazon RDS MySQL stores metadata about uploaded notes, including information such as:

- Note ID
- Course name
- File name
- Description
- S3 object key
- Upload information

The database is deployed in private subnets and is not publicly accessible.

### S3 Storage

Amazon S3 stores the actual uploaded PDF files.

- Bucket: `campus-notes-efrata-2026-mira`
- Bucket access: Private
- Block Public Access: Enabled
- Object ownership: Bucket owner enforced
- Server-side encryption: Enabled

Uploaded objects are stored under the `notes/` prefix.

The application uses its EC2 IAM role to access S3 instead of storing static AWS access keys.

---

## 6. Security Configuration

Security measures implemented in the project include:

- VPC network isolation.
- Public and private subnet separation.
- RDS database deployed privately.
- Database access restricted to the application security group.
- S3 Block Public Access enabled.
- S3 server-side encryption.
- IAM role attached to EC2.
- Least-privilege S3 permissions for the application.
- AWS Systems Manager Session Manager for server access.
- Environment secrets stored outside the repository.
- Restricted permissions on the environment configuration file.
- CloudTrail used to review AWS management events.
- CloudWatch alarm configured to monitor EC2 CPU utilization.

---

## 7. Evidence

Screenshots are provided in the `screenshots/` directory.

Evidence includes:

- AWS VPC and subnet configuration.
- Route tables and network configuration.
- S3 bucket security settings.
- IAM role and policy.
- EC2 instance configuration.
- RDS database configuration.
- Application running in the browser.
- Upload and download testing.
- Database verification.
- S3 object verification.
- CloudWatch alarm testing.
- CloudTrail event verification.
- Troubleshooting exercises.

---

## 8. Known Limitations, Cost and Cleanup

### Known Limitations

- The application does not currently have user authentication.
- Anyone who can access the application can upload and download notes.
- The application uses HTTP rather than HTTPS.
- Users cannot delete or edit uploaded notes through the application.
- Upload rate limiting and CSRF protection are not implemented.
- If metadata insertion fails after a file is uploaded, the S3 object may remain.
- The current application is intended for a coursework demonstration and not production use.

### Cost Considerations

The project uses AWS resources that may incur charges, including:

- EC2
- Amazon RDS
- Amazon S3
- CloudWatch

Actual costs depend on resource usage, region, service configuration, and applicable free-tier eligibility. AWS Budgets was configured to help monitor spending.

### Cleanup

After completing the project demonstration and saving all required evidence, AWS resources should be reviewed and deleted when they are no longer needed to avoid continued charges.

Resources to review include:

- EC2 instance
- RDS database
- S3 bucket and uploaded objects
- VPC and related networking resources
- CloudWatch alarms
- IAM resources created specifically for this project

Resources should not be deleted until the final presentation and required evidence have been completed.
