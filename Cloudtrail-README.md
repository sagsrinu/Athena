# AWS CloudTrail Logs Querying Using Athena

## Step 1: Create CloudTrail

1. Open AWS Console
2. Search for:

```text
CloudTrail
```

3. Click:

```text
Trails
```

4. Click:

```text
Create trail
```

---

# Step 2: Configure Trail

## Create New S3 Bucket

While creating the trail:

- Create a new S3 bucket
- Provide bucket name

---

## Encryption Settings

Untick:

```text
Log file SSE-KMS encryption
```

Keep Trail Status:

```text
Enabled
```

---

# Step 3: Select Event Types

Select all three event types:

- Management events
- Data events
- Insights events

---

# Step 4: Configure Data Events

Under:

```text
Data event: S3
```

Select:

```text
Resource type → S3
```

Then select:

```text
All current and future S3 buckets
```

---

# Step 5: Configure Insights Events

Enable all Insights events.

## Management Events Insights Types

Select:

- API call rate
- API error rate

### API Call Rate

Measures write-only management API calls occurring per minute against a baseline API call volume.

### API Error Rate

Measures management API calls that return errors.

---

## Data Events Insights Types

Select:

- API call rate
- API error rate

### Data API Call Rate

Measures data API calls occurring per minute against a baseline API call volume.

### Data API Error Rate

Measures data API calls resulting in errors.

> Additional charges apply for Insights events.

---

# Step 6: Aggregation Templates

Under:

```text
Aggregation templates
```

Select:

```text
All
```

---

# Step 7: Open Athena

1. Open Athena
2. Click:

```text
Query Editor
```

---

# Step 8: Create Database

Run the following query:

```sql
CREATE DATABASE cloudtrail_logs_db;
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

# Step 9: Create External Table

> Change the S3 bucket name and AWS account ID in the LOCATION path.

```sql
CREATE EXTERNAL TABLE cloudtrail_logs (
  Records ARRAY<
    STRUCT<
      eventVersion:STRING,
      userIdentity:STRUCT<
        type:STRING,
        principalId:STRING,
        arn:STRING,
        accountId:STRING,
        accessKeyId:STRING,
        userName:STRING,
        sessionContext:STRUCT<
          attributes:STRUCT<
            creationDate:STRING,
            mfaAuthenticated:STRING
          >,
          sessionIssuer:STRUCT<
            type:STRING,
            principalId:STRING,
            arn:STRING,
            accountId:STRING,
            userName:STRING
          >
        >
      >,
      eventTime:STRING,
      eventSource:STRING,
      eventName:STRING,
      awsRegion:STRING,
      sourceIPAddress:STRING,
      userAgent:STRING,
      errorCode:STRING,
      errorMessage:STRING,
      requestParameters:STRING,
      responseElements:STRING,
      additionalEventData:STRING,
      requestID:STRING,
      eventID:STRING,
      readOnly:STRING,
      eventType:STRING,
      managementEvent:STRING,
      recipientAccountId:STRING,
      eventCategory:STRING
    >
  >
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
LOCATION 's3://aws-cloudtrail-logs-339712994180-e514d4c4/AWSLogs/339712994180/CloudTrail/';
```

---

# Sample Athena Queries

## Show All Events

```sql
SELECT *
FROM cloudtrail_logs
CROSS JOIN UNNEST(Records) AS t(record)
LIMIT 50;
```

---
# SPECIFIC DATE 
```
SELECT
  record.eventTime,
  record.eventSource,
  record.eventName,
  record.sourceIPAddress
FROM cloudtrail_logs
CROSS JOIN UNNEST(Records) AS t(record)
WHERE date(from_iso8601_timestamp(record.eventTime))
      = DATE '2026-05-23'
ORDER BY record.eventTime DESC;
```
---
# time based like last 15 min 40 min

```
SELECT
  record.eventTime,
  record.eventSource,
  record.eventName,
  record.sourceIPAddress
FROM cloudtrail_logs
CROSS JOIN UNNEST(Records) AS t(record)
WHERE from_iso8601_timestamp(record.eventTime)
      >= current_timestamp - interval '15' minute
ORDER BY record.eventTime DESC;
```
---

# Show Recent API Calls

```sql
SELECT
  record.eventTime,
  record.eventSource,
  record.eventName,
  record.awsRegion
FROM cloudtrail_logs
CROSS JOIN UNNEST(Records) AS t(record)
ORDER BY record.eventTime DESC
LIMIT 100;
```

---

# Search EC2 Events

```sql
SELECT
  record.eventTime,
  record.eventName,
  record.sourceIPAddress
FROM cloudtrail_logs
CROSS JOIN UNNEST(Records) AS t(record)
WHERE record.eventSource = 'ec2.amazonaws.com'
LIMIT 100;
```

---

# Search S3 Bucket Activity

```sql
SELECT
  record.eventTime,
  record.eventName,
  record.sourceIPAddress
FROM cloudtrail_logs
CROSS JOIN UNNEST(Records) AS t(record)
WHERE record.eventSource = 's3.amazonaws.com'
LIMIT 100;
```

---

# Search Failed Events

```sql
SELECT
  record.eventTime,
  record.eventSource,
  record.eventName,
  record.errorCode,
  record.errorMessage
FROM cloudtrail_logs
CROSS JOIN UNNEST(Records) AS t(record)
WHERE record.errorCode IS NOT NULL
LIMIT 100;
```

---

# Notes

- Ensure CloudTrail is enabled.
- Ensure S3 bucket permissions are correct.
- Athena automatically scans logs stored in the specified S3 path.
- Replace:
  - S3 bucket name
  - AWS account ID
  before running the table creation query.

Example:

```sql
LOCATION 's3://your-cloudtrail-bucket/AWSLogs/ACCOUNT-ID/CloudTrail/';
``` 
