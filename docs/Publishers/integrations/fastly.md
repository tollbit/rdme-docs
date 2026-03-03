---
title: Fastly
excerpt: Learn how to integrate TollBit with Fastly.
---
There are two integration options to connect Fastly with TollBit. 

* The Standard method uses API tokens to give TollBit access to read http traffic logs and implement bot paywall directly through the TollBit interface.
* The Custom method uses Fastly's HTTPS endpoint and VCL scripts (directly within the Fastly navigation) to implement both logging and paywall as well.

# Fastly (Standard)

Follow these steps to set up an integration into our platform if you use Fastly.

**Get Service ID and API Key from Fastly**

Go to your Fastly Dashboard and pick the domain associated with your property.

Right under your service name, you’ll see an alphanumeric string. It should be the same alphanumeric string that completes the URL string for the page. See the highlight below for reference.

![Integrations Fastly Service](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/integrations-fastly-service.png)

Next, hover over Account on the main navigation bar on the left and choose API tokens > personal tokens.

![Integrations Fastly Service 2](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/integrations-fastly-service-2.png)

**Setup Integration in TollBit**

Go to your TollBit dashboard and pick the Integrations tab in the main navigation menu. Input your Fastly API key and service ID in the form and click Save.

![Integrations Fastly Service 3](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/integrations-fastly-service-3.png)

### Enable Analytics

Ensure that you have saved your Fastly API key and service ID in the integration settings. Once that is saved, within the same page, click on “Enable” next to the Analytics section.

![Integrations Fastly Service 4](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/integrations-fastly-service-4.png)

### Enable Bot Paywall

Ensure that you have saved your Fastly API key and service ID in the integration settings. Once that is saved, within the same page, toggle on “Block” for each agent you would like to forward to your TollBit subdomain.

Scrolling further down on the page allows you to “Block” all bots, which would redirect all listed bots on the page to forward to your TollBit subdomain.

![Integrations Fastly Service 5](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/integrations-fastly-service-5.png)

<Callout icon="📘" theme="info">
  If you have used our legacy integration of Fastly (using VCL scripts), you should automatically see the updates transition into the new UI.
</Callout>

## Fastly (Custom)

This is the documentation for the legacy Fastly integration that involves implementing VCL scripts to enable TollBit analytics and bot paywall. VCL scripts can allow for additional customizations for implementing analytics and bot forwarding. Please reach out to [team@tollbit.com](mailto:team@tollbit.com) if you'd like to discuss this implementation route considering your use case.

### Enable Analytics

**Create a new Logging Configuration**

Go to your <Anchor label="Fastly Dashboard" target="_blank" href="https://manage.fastly.com/configure">Fastly Dashboard</Anchor> and pick the
correct domain. Click “Edit Configuration”, and clone your current
configuration. This saves a new configuration version as a draft, and allows you
to rollback if necessary. This should bring you to a new screen. On the sidebar,
scroll down until you see Logging and click on that. Then, click “Create
Endpoint”.

![Fastly Sidebar](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-sidebar.png)

** Configure your logs to be sent to our logging endpoint **

![Fastly Http Config](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-http-config.png)

Find the HTTP logging endpoint and click “Create endpoint”. You can set the name
to anything descriptive (e.g. tollbit-prod). Keep the placement option as the
default selection. Make sure your log format is exactly as follows, without
extra trailing spaces or newlines:

```
{ "timestamp": "%{strftime(\{"%Y-%m-%dT%H:%M:%S%z"\}, time.start)}V", "ip_address": "%{req.http.Fastly-Client-IP}V", "geo_country": "%{client.geo.country_name}V", "geo_city": "%{client.geo.city}V", "geo_postal_code":"%{client.geo.postal_code}V", "geo_latitude":"%{client.geo.latitude}V", "geo_longitude":"%{client.geo.longitude}V", "host": "%{if(req.http.Fastly-Orig-Host, req.http.Fastly-Orig-Host, req.http.Host)}V", "url": "%{json.escape(req.url)}V", "request_method": "%{json.escape(req.method)}V", "request_protocol": "%{json.escape(req.proto)}V", "request_referer": "%{json.escape(req.http.referer)}V", "request_user_agent": "%{json.escape(req.http.User-Agent)}V", "request_latency":"%{time.elapsed.usec}V", "response_state": "%{json.escape(fastly_info.state)}V", "response_status": %{std.itoa(resp.status)}V, "response_reason": %{if(resp.response, "%22"+json.escape(resp.response)+"%22", "null")}V, "response_body_size": %{resp.body_bytes_written}V, "fastly_server": "%{json.escape(server.identity)}V", "fastly_is_edge": %{if(fastly.ff.visits_this_service == 0, "true", "false")}V, "signature": "%{json.escape(req.http.signature)}V", "signature_agent": "%{json.escape(req.http.signature-agent)}V", "signature_input": "%{json.escape(req.http.signature-input)}V" }
```

Finally, set the URL to [https://log.tollbit.com/log](https://log.tollbit.com/log).

![Fastly Log Config](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-log-config.png)

** Ensure that your Requests are Authenticated **

Go into Advanced Options and set the “Custom header name” field to “TollbitKey”.
You must set the customer header value to your secret key. Log into your
<Anchor label="TollBit portal" target="_blank" href="https://app.tollbit.com">TollBit portal</Anchor> and go into the API key tab and copy
your secret key. Paste it into the “Custom header value” field with no trailing
spaces. Keep all the other settings as default, scroll to the bottom, and save.

![Fastly Activate](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-activate.png)

Once you are ready to publish these changes, click the “Activate” button. Keep
in mind that if you have other unpublished changes in Fastly, this may also
publish those as well.

### Enable Bot Paywall

Fastly allows you to set up redirectly using VCL snippets. In this document, we will go over setting up forwarding requests from known bots to your tollbit subdomain.

<Callout icon="📘" theme="info">
  The code shown here is for a clean Fastly environment. If you have any
  existing VCL scripts that intercept requests, you will need to integrate these
  scripts into your existing workflow.
</Callout>

Go to the Deliver tab and select the domain you wish to add bot forwarding to. On the right side of the screen, click the Edit configuration button and choose to clone your current active version.

![Fastly Edit Configuration](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-edit-configuration.png)

On the left hand sidebar, click "VCL Snippets".

![Fastly Vcl Snippet](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-vcl-snippet.png)

Create a snippet and name it something like `tollbit-bot-forwarding-recv`. This is the VCL code that will detect if a bot is using one of our known bad user agents, and will forward it to your subdomain. Put the following logic into the snippet. Make sure that the placement of the snippet is within the recv subroutine.

![Fastly Recv](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-recv.png)

Copy and paste the following code block into the VCL input field and save. Don't worry, this VCL script will not actually apply until you activate the current Fastly version that you are editing.

```
if (req.http.user-agent ~ "(?i)chatgpt-user|perplexitybot|gptbot|anthropic-ai|ccbot|claude-web|claudebot|cohere-ai|youbot|diffbot|oai-searchbot|meta-externalagent|timpibot|amazonbot|bytespider|perplexity-user") {
  if (std.prefixof(req.http.host, "www.")) {
    set req.http.host = std.replace_prefix(req.http.host, "www.", "tollbit.");
  } else {
    set req.http.host = "tollbit." + req.http.host;
  }
  error 600;
}
```

Next, create another VCL snippet. This time, call it something like `tollbit-bot-forwarding-error`. This time, make sure that the placement is within the error subroutine.

![Fastly Error](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-error.png)

Paste the following code in this snippet. This will set the correct headers and status code for the redirection done in the previous snippet.

```
if (obj.status == 600) {
  set obj.status = 307;
  set obj.response = "Temporary Redirect";
  set obj.http.Location = req.protocol + "://" req.http.host + req.url;
  set obj.http.cache-control = "max-age=0";
  return (deliver);
}
```

<Callout icon="🚧" theme="warn">
  The VCL scripts you just added **will intercept** and potentially **redirect**
  traffic to your main site. Please ensure that you have tested this in a test
  environment or for a small subset of pages before activating this across your
  entire site.
</Callout>

This should now be all you need to forward known bot traffic to your `tollbit` subdomain! You can activate these changes by clicking "Apply".

![Fastly Activate](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-activate.png)
