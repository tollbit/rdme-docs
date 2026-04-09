---
title: WordPress VIP
excerpt: Learn how to integrate TollBit with WordPress VIP.
---
# WordPress VIP

The integration supports TollBit's AI bot monitoring, management and monetization capabilities as well as seamless connections with MCP and NLWeb.

Follow these steps to set up an integration into our platform if you use WordPress VIP.

<br />

### Setup Analytics with WordPress VIP

TollBit can ingest http traffic logs from Wordpress VIP using [WP Log Shopping](https://docs.wpvip.com/logs/log-shipping/enable/).

Log in to the WordPress VIP Dashboard and select environment from the environment dropdown located at the upper left of the VIP Dashboard. Select "Log Shipping" under the "Logs" navigation bar on the left.

Now you can configure the Cloud Configuration. Note you'll need a storage bucket (S3, GCS, or Azure) with the correct IAM permissions (see tab for General Log Ingestion) to export the logs from the Log Shipping tool.

<br />

PREFER S3 UPDATE

![](https://files.readme.io/6ecc0147e56563dee287f99f546c9826961fc8e7f6de45d165763f75e948723c-image.png)

Please reach out to [team@tollbit.com](mailto:team@tollbit.com) when the storage bucket is provisioned with the log file data and the IAM policy.

<br />

### Route to Agent Site in WordPress VIP

Log in to the WordPress VIP Dashboard and select the banner navigation menu item labeled “Integrations Center” at the top of the VIP Dashboard.

Select TollBit from the Integrations Center list to access the information page for TollBit. An overview of the Integration’s features and functionality is provided, as well as information about the publisher and links to support documentation.
To add the Integration to an organization, select the button labeled “+ Add to Organization” located at the top of the right-hand column.
If the current user has an Org admin role for more than one organization, the organization to which the Integration will be added must be selected from the auto-generated option dropdown after the button is selected.

![Integrations Wordpress Service 1](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/integrations-wordpress-service-1.png)
