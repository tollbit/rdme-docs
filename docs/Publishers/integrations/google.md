---
title: Google (GCP)
excerpt: Learn how to integrate TollBit with Google.
---
### Setting up Logging with CDN/Cloud Load Balancer

To set up logging for your Google Cloud Load Balancer instance (if you are using Google CDN, it should be backed
by a Cloud Load Balancer), you can forward logs to a GCP Storage Bucket.

First, create a bucket you would like to use to hold the logs.

![Google Alb Create Bucket Flow](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/google-alb-create-bucket-flow.png)

If your load balancer is backed by a non-static backend (you are using another domain or IP address as
an orgin, and not a Storage Bucket), you may need to edit your load balancer's configs and enable a 100%
sampling rate for backend logging.

Next, go to the Log Explorer page and on the left hand nav bar, click into "Log router".

![Google Alb Create Router](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/google-alb-create-router.png)

On the top bar, click "Create sink".

![Google Alb Create Sink](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/google-alb-create-sink.png)

Go through the sink creation flow, making sure to set the Storage Bucket you created earlier as
the destination. You should set an inclusion filter to ensure that only traffic logs for your load balancer
gets stored. Some fields to use for the inclusion filter could be the ID of the load balancer,
the underlying domain or IP address, the URL that routes to the load balancer, etc.

Once this sink is created, you may need to wait up to an hour for logs to start appearing in your bucket.
Once you've verified that this is set up correctly, please contact us at [team@tollbit.com](mailto:team@tollbit.com)
to share your bucket with us.

### Google Cloud Armor Route to Agent Site

Google's Cloud Armor allows you to set up some simple redirection rules for user agents.

Note that to implement the full solution, where we want to preserve the path of the content, you
will need to set up a separate backend service that handles redirection that preserves path. However,
you can simply just redirect to the root `tollbit` subdomain as well to get most of the functionaltiy.

First, navigate
to Cloud Armor policies and create a new one (or add this to your existing policy). Set the default rule to
allow.

![Google Cloud Armor Create Policy](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/google-cloud-armor-create-policy.png)

Next, add more rules and select "Advanced mode". You can add preferred user agents that you want to
redirect in the match rules box.

Next, select "Redirect" as the action for the rule, and if you do have a redirection backend service that
preserves path, put the URL to that service. Otherwise, put the root `tollbit` subdomain for your site (`tollbit.yoursite.com`).

![Google Cloud Armor Redirect](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/google-cloud-armor-redirect.png)

Save and activate your policy.
