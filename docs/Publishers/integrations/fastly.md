---
title: Fastly
excerpt: Learn how to integrate TollBit with Fastly.
---
There are two integration options to connect Fastly with TollBit.

* The Custom method uses Fastly's HTTPS endpoint and VCL scripts (directly within the Fastly navigation) to implement both logging and agent site routing.
  * This is the recommended route for teams with multiple Fastly services and/or advanced Infrastructure as code workflows.
* The TollBit-Managed method uses API tokens to give TollBit access to read HTTP traffic logs and implement agent site routing directly through the TollBit interface.

## Fastly (Custom)

This is the documentation for the Fastly integration that involves implementing VCL scripts to enable TollBit analytics and agent site routing. VCL scripts allow for additional customizations for implementing analytics and bot forwarding. Please reach out to [team@tollbit.com](mailto:team@tollbit.com) if you'd like to discuss this implementation route considering your use case.

### Enable Analytics

**Create a new Logging Configuration**

Go to your <Anchor label="Fastly Dashboard" target="_blank" href="https://manage.fastly.com/configure">Fastly Dashboard</Anchor> and pick the
correct domain. Click "Edit Configuration", and clone your current
configuration. This saves a new configuration version as a draft, and allows you
to rollback if necessary. This should bring you to a new screen. On the sidebar,
scroll down until you see Logging and click on that. Then, click "Create
Endpoint".

![Fastly Sidebar](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-sidebar.png)

** Configure your logs to be sent to our logging endpoint **

![Fastly Http Config](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-http-config.png)

Find the HTTP logging endpoint and click "Create endpoint". You can set the name
to anything descriptive (e.g. `tollbit-prod`). Keep the placement option as the
default selection. Make sure your log format is exactly as follows, without
extra trailing spaces or newlines:

```
{ "timestamp": "%{strftime(\{"%Y-%m-%dT%H:%M:%S%z"\}, time.start)}V", "ip_address": "%{req.http.Fastly-Client-IP}V", "geo_country": "%{client.geo.country_name}V", "geo_city": "%{client.geo.city}V", "geo_postal_code":"%{client.geo.postal_code}V", "geo_latitude":"%{client.geo.latitude}V", "geo_longitude":"%{client.geo.longitude}V", "host": "%{if(req.http.Fastly-Orig-Host, req.http.Fastly-Orig-Host, req.http.Host)}V", "url": "%{json.escape(req.url)}V", "request_method": "%{json.escape(req.method)}V", "request_protocol": "%{json.escape(req.proto)}V", "request_referer": "%{json.escape(req.http.referer)}V", "request_user_agent": "%{json.escape(req.http.User-Agent)}V", "request_latency":"%{time.elapsed.usec}V", "response_state": "%{json.escape(fastly_info.state)}V", "response_status": %{std.itoa(resp.status)}V, "response_reason": %{if(resp.response, "%22"+json.escape(resp.response)+"%22", "null")}V, "response_body_size": %{resp.body_bytes_written}V, "fastly_server": "%{json.escape(server.identity)}V", "fastly_is_edge": %{if(fastly.ff.visits_this_service == 0, "true", "false")}V, "signature": "%{json.escape(req.http.signature)}V", "signature_agent": "%{json.escape(req.http.signature-agent)}V", "signature_input": "%{json.escape(req.http.signature-input)}V" }
```

Finally, set the URL to [https://log.tollbit.com/log](https://log.tollbit.com/log).

![Fastly Log Config](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-log-config.png)

** Ensure that your Requests are Authenticated **

Go into Advanced Options and set the "Custom header name" field to "TollbitKey".
You must set the customer header value to your secret key. Log into your
<Anchor label="TollBit portal" target="_blank" href="https://app.tollbit.com">TollBit portal</Anchor> and go into the API key tab and copy
your secret key. Paste it into the "Custom header value" field with no trailing
spaces. Keep all the other settings as default, scroll to the bottom, and save.

![Fastly Activate](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-activate.png)

Once you are ready to publish these changes, click the "Activate" button. Keep
in mind that if you have other unpublished changes in Fastly, this may also
publish those as well.

### Route to Agent Site

Fastly allows you to set up an internal rewrite using VCL snippets. In this section, we will go over routing requests from known bots directly to your `tollbit` subdomain backend—without issuing a redirect to the bot. Because many AI scrapers ignore 3xx redirects, this approach ensures every bot hit reaches TollBit for metering and access control.

<Callout icon="📘" theme="info">
  The code shown here is for a clean Fastly environment. If you have any
  existing VCL scripts that intercept requests, you will need to integrate these
  scripts into your existing workflow.
</Callout>

**Create a Tollbit Backend in Fastly**

Before updating your VCL, add Tollbit as an origin (backend) in your Fastly service configuration.

Go to your Fastly Dashboard and select the service for your domain. Click "Edit Configuration" and clone your current active version. On the left sidebar, click "Origins" and then click "Create a host".

<!-- TODO: add screenshot of Fastly Origins / Create a host screen -->

Configure the new host with the following settings:

- **Name**: Give it a descriptive name such as `Tollbit Backend`. Fastly will generate a VCL name in the format `F_<Name_With_Underscores>` (e.g. `F_Tollbit_Backend`). Note this name—you will use it in the VCL snippet below.
- **Address**: Set this to the Tollbit gateway address for your domain (e.g. `tollbit.yourdomain.com`).
- **Override Host**: Leave this blank so the dynamically computed `tollbit.` host header set in your VCL is passed through correctly.

<!-- TODO: add screenshot of backend host configuration fields -->

**Add the VCL recv Snippet**

Go to the Deliver tab and select the domain you wish to add bot forwarding to. On the right side of the screen, click the Edit configuration button and choose to clone your current active version.

![Fastly Edit Configuration](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-edit-configuration.png)

On the left hand sidebar, click "VCL Snippets".

![Fastly Vcl Snippet](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-vcl-snippet.png)

Create a snippet and name it something like `tollbit-bot-forwarding-recv`. This VCL code detects known bot user agents, rewrites the host to your `tollbit` subdomain, and routes the request directly to the Tollbit backend without issuing a redirect. Make sure the placement of the snippet is within the recv subroutine.

![Fastly Recv](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-recv.png)

Copy and paste the following code block into the VCL input field and save. Replace `F_Tollbit_Backend` with the VCL name Fastly generated for the backend you created above. Don't worry, this VCL script will not actually apply until you activate the current Fastly version that you are editing.

```
if (req.http.user-agent ~ "(?i)chatgpt-user|perplexitybot|gptbot|anthropic-ai|ccbot|claude-web|claudebot|cohere-ai|youbot|diffbot|oai-searchbot|meta-externalagent|timpibot|amazonbot|bytespider|perplexity-user") {
  if (std.prefixof(req.http.host, "www.")) {
    set req.http.host = std.replace_prefix(req.http.host, "www.", "tollbit.");
  } else {
    set req.http.host = "tollbit." + req.http.host;
  }
  set req.backend = F_Tollbit_Backend;
  return(pass);
}
```

The `return(pass)` directive bypasses Fastly's cache for these requests, ensuring Tollbit sees and meters every bot hit.

<Callout icon="🚧" theme="warn">
  The VCL script you just added **will intercept** and potentially **rewrite**
  traffic to your main site. Please ensure that you have tested this in a test
  environment or for a small subset of pages before activating this across your
  entire site.
</Callout>

This should now be all you need to forward known bot traffic to your `tollbit` subdomain! You can activate these changes by clicking "Apply".

![Fastly Activate](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-activate.png)

## Fastly (TollBit-Managed)

Follow these steps to set up an integration into our platform if you use Fastly.

### Initial Setup for Analytics & Agent Site Routing

**Get Service ID and API Key from Fastly**

Go to your Fastly Dashboard and pick the domain associated with your property.

Right under your service name, you'll see an alphanumeric string. It should be the same alphanumeric string that completes the URL string for the page. See the highlight below for reference.

![Integrations Fastly Service](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/integrations-fastly-service.png)

Next, hover over Account on the main navigation bar on the left and choose API tokens > personal tokens.

![Integrations Fastly Service 2](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/integrations-fastly-service-2.png)

**Setup Integration in TollBit**

Go to your TollBit dashboard and pick the Integrations tab in the main navigation menu. Input your Fastly API key and service ID in the form and click Save.

![Integrations Fastly Service 3](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/integrations-fastly-service-3.png)

### Enable Analytics

Ensure that you have saved your Fastly API key and service ID in the integration settings. Once that is saved, within the same page, click on "Enable" next to the Analytics section.

![Integrations Fastly Service 4](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/integrations-fastly-service-4.png)

### Route to your Agent Site

Ensure that you have saved your Fastly API key and service ID in the integration settings. Once that is saved, within the same page, toggle on "Block" for each agent you would like to forward to your `tollbit` subdomain.

Scrolling further down on the page allows you to "Block" all bots, which would redirect all listed bots on the page to forward to your `tollbit` subdomain.

![Integrations Fastly Service 5](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/integrations-fastly-service-5.png)

<Callout icon="📘" theme="info">
  If you have used our legacy integration of Fastly (using VCL scripts), you should automatically see the updates transition into the new UI.
</Callout>
