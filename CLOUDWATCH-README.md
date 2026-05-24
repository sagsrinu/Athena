# Athena - Query CloudWatch Logs from S3

## Step 1: Create EC2 Instance

Create an EC2 instance and attach an IAM Role with:

- S3 Access
- CloudWatch Access

Recommended Policies:

- `AmazonS3FullAccess`
- `CloudWatchAgentServerPolicy`

---

# Step 2: Create S3 Bucket

Create an S3 bucket to store exported CloudWatch logs.

Example:

```bash
logs-s33-vsv
```

---

# Step 3: Install CloudWatch Agent

Install the CloudWatch agent on the EC2 instance.

```bash
sudo yum install amazon-cloudwatch-agent -y
```

---

# Step 4: Configure CloudWatch Agent

Create the configuration file:

```bash
sudo vi /opt/aws/amazon-cloudwatch-agent/bin/config.json
```

Add the following configuration:

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/*",
            "log_group_name": "ec2-all-logs",
            "log_stream_name": "{instance_id}-all-logs"
          }
        ]
      }
    }
  }
}
```

---

# Step 5: Start CloudWatch Agent

Start the agent using the configuration file:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
-a fetch-config \
-m ec2 \
-c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
-s
```

---

# Step 6: Install HTTPD

Install Apache HTTP Server:

```bash
sudo yum install httpd -y
```

Start and enable the service:

```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

Check status:

```bash
sudo systemctl status httpd
```

---

# Step 7: Add S3 Bucket Policy

Go to:

```text
S3 Bucket → Permissions → Bucket Policy
```

Add the following policy.

> Change the bucket name in the policy.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCloudWatchLogs",
            "Effect": "Allow",
            "Principal": {
                "Service": "logs.amazonaws.com"
            },
            "Action": [
                "s3:PutObject",
                "s3:GetBucketAcl",
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::my-cloudwatch-logs-bucket-pr/*",
                "arn:aws:s3:::my-cloudwatch-logs-bucket-pr"
            ]
        }
    ]
}
```

---

# Step 8: Export CloudWatch Logs to S3

1. Open AWS Console
2. Go to CloudWatch
3. Open Log Groups
4. Select your log group
5. Click Actions
6. Select:

```text
Export data to Amazon S3
```

7. Choose:
   - S3 Bucket
   - Start Time
   - End Time

> While exporting logs, check the time in UTC format.

---

# Step 9: Open Athena

1. Go to Athena
2. Click Query Editor

---

# Step 10: Create Database

Run the following query:

```sql
CREATE DATABASE cloudwatch_logs_db;
```
### if no un is fade out mens do this process
```
Go like this in Athena:

Open Amazon Athena Console
Click Settings (top right)
Under Manage settings → Query result location
Enter your bucket path like:
s3://logs-s33-vsv/athena-results/
```
---

# Step 11: Create External Table

> Change the S3 bucket path with your bucket details.

```sql
CREATE EXTERNAL TABLE cloudwatch_logs (
  line string
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\n'
LOCATION 's3://logs-s33-vsv/exportedlogs/';
```

---

# Sample Athena Queries

## Count Total Logs

```sql
SELECT count(*) FROM cloudwatch_logs;
```

---

## Search HTTPD Logs

```sql
SELECT *
FROM cloudwatch_logs
WHERE lower(line) LIKE '%httpd%'
LIMIT 100;
```

---

## Search System Logs

```sql
SELECT *
FROM cloudwatch_logs
WHERE lower(line) LIKE '%system%'
LIMIT 100;
```

---

## Search Start Logs

```sql
SELECT *
FROM cloudwatch_logs
WHERE lower(line) LIKE '%start%'
LIMIT 100;
```

---
## date wise logs fetching
```
SELECT *
FROM cloudwatch_logs
WHERE line LIKE '%2026-05-23%'
LIMIT 100;
```
---
### date and time rannge
```
SELECT *
FROM cloudwatch_logs
WHERE line LIKE '%2026-05-23%'
AND (
      line LIKE '%17:%'
   OR line LIKE '%18:%'
)
LIMIT 100;
```

## Search YUM Logs

```sql
SELECT *
FROM cloudwatch_logs
WHERE lower(line) LIKE '%yum%'
LIMIT 100;
```

---

# Notes

- Ensure CloudWatch Agent is running.
- Ensure the IAM Role has:
  - S3 Access
  - CloudWatch Access
- Ensure the S3 bucket policy is added correctly.
- Athena automatically scans all folders inside the specified S3 path.

Example:

```sql
LOCATION 's3://logs-s33-vsv/exportedlogs/';
```

This will read logs from all folders inside `exportedlogs/`.

```
