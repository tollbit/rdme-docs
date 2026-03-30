---
title: General Log Ingestion
excerpt: Learn how to integrate TollBit Analytics with Other Log Ingestion Methods.
---
# Other Integration Methods

We support other methods of log ingestion besides the integrations that we
listed above.

<br />

### Ingesting from file storage

We are currently able to support log ingestion from S3, R2 and GCS. Please
ensure that your log files are prefixed by date and time, and that the logs
within the files are in JSON format (ideally as similar to the above as
possible), and each log is a single line and all logs are newline separated.

If you are already forwarding logs to an S3 bucket, you can get a headstart on
the setup by creating the following IAM policy for your bucket: If your logs are
already being sent to an S3 bucket, add the following IAM policy to your bucket
to enable TollBit to process your logs:

```json
{
  "Version": "2025-05-07",
  "Statement": [
    {
      "Sid": "AllowTollbitAccountsAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::339712821696:root",
          "arn:aws:iam::654654318267:root"
        ]
      },
      "Action": ["s3:GetObject*", "s3:ListBucket*"],
      "Resource": [
        "arn:aws:s3:::YOUR-BUCKET-NAME",
        "arn:aws:s3:::YOUR-BUCKET-NAME/*"
      ]
    }
  ]
}
```

#### Buckets with ACLs (Access Control Lists)

The AWS recommendation is to disable ACLs and instead fully rely on IAM policies to control bucket access. However, there may be some legacy workflows that still requires ACLs, or at least some investigation into whether they can be safely disabled. 

In case your AWS bucket has any Access Control Lists (ACLs) added, we recommend those. If you still wish to keep your bucket ACLs, we recommend first adding our canonical IDs to the bucket:

TollBit Canonical IDs:

```json
f208f3b6492f906045d6e2fd71dbc6dc64247211c059ba3ae2eb712b949b8cdd
bbfe4f5f203ffb85e03d5d5325ab889271d6429b91a3766563bd0d08df2126ab
```

and then update the object ownership permissions to "Bucket Owner Enforced". This should make your bucket the owner of objects written to it, which would allow us to read new objects written going forward, as they should respect your bucket's IAM policy.

<Image align="center" src="https://files.readme.io/f2801981eda231d5b01ca3c2b819f3ec727bc1d95cc848ceddf3da962b23a0c1-Screenshot_2026-03-30_at_10.40.00_AM.png" />

Please contact us at [team@tollbit.com](mailto:team@tollbit.com)  to complete the set up for you so you can
get access to TollBit Analytics

<br />

### Log Sink Forwarding

You can forward your logs to our log sink endpoint at
`https://log.tollbit.com/log` as long as you include the header `TollbitKey` and
set the value to your secret key in your dashboard. The logs must conform to the
following JSON format. Not all fields are required, but we need at least the
`timestamp`, `host`, `url`, `request_user_agent`, `response_status`, `request_referer` and `request_method`.

```json
{
  timestamp: string, // can be ISO 8601 format or unix timestamp
  geo_country: string,
  geo_city: string,
  geo_postal_code: string,
  geo_latitude: float,
  geo_longitude: float,
  host: string,
  url: string,
  request_method: string,
  request_protocol: string,
  request_user_agent: string,
  request_latency: int/string,
  request_referer: string,
  response_state: string,
  response_status: int/string,
  response_reason: string,
  response_body_size: int/string,
  signature: string,
  signature_agent: string,
  signature_input: string,
  ip_address: string
}
```

When streaming the logs to the endpoint, please ensure that you are batching
logs as much as possible. Each log be a single line, and should be newline
separated from the other logs.

<br />
