---
title: Other Integration Methods
excerpt: Learn how to integrate TollBit with Other Integration Methods.
---
# Other Integration Methods


We support other methods of log ingestion besides the integrations that we
listed above.

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

Please contact us at team@tollbit.com to complete the set up for you so you can
get access to TollBit Analytics