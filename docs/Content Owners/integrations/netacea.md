---
title: Netacea
excerpt: Learn how to implementat TollBit with Netacea
deprecated: false
hidden: false
metadata:
  robots: index
---
# Allow TollBit's IP Addresses

<br />

<Callout icon="🚧" theme="warn">
  ### Required

  TollBit must be able to reach your sites. We request your pages to set up your Agent Site. If our requests are blocked, challenged, or rate-limited, your Agent Site will not work correctly.

  TollBit sends these requests from a fixed set of static IP addresses, published at [https://tollbit.com/static-ips.txt](https://tollbit.com/static-ips.txt). At the time of writing, the list is:
</Callout>

TollBit sends these requests from a fixed set of static IP addresses, published at [https://tollbit.com/static-ips.txt](https://tollbit.com/static-ips.txt). At the time of writing, the list is:

- 52.22.183.94
- 3.220.109.109


Always use the published list as the source of truth, and check it again if TollBit tells you it has changed.

Our requests use the user agent tollbot/1.0, so you may see it show up in your logs.

Make sure these addresses are allowed through every security layer that could block or challenge our requests. This applies to every hostname you onboard with us:

- WAF: the rules in your Web ACL, including IP set rules, rate-based rules, and managed rule groups such as Bot Control.
- Any firewall or WAF in front of your origin servers, including security groups and any protection outside of AWS.

If you are unsure whether your configuration blocks us, contact [support@tollbit.com](mailto:team@tollbit.com) and we will confirm from our side.

# Steps for Analytics

Reach out to your Netacea Customer Sucess Manager. Let them know you'd like the TollBit analytics integration switched on, and they'll update your account with the right features to get started.

For enabling analytics specifically, please see the 'General Route' tab to enable site-wide analytics.

# Steps for Agent Site

Go to the Known Bots LLMs dashboard within your Netacea portal.&#x20;


<Image src="https://files.readme.io/e47eb4231b92a7e49f428022630f086cb02628cc5adad6b45d81e86ccc2d73e3-Screenshot_2026-10-20_at_3.56.20_PM.png" width="100px" />


<br />

On the actions menu from the table you will have a **Monetize** option for each agent under the list.

![](https://files.readme.io/76b4854cb3a1f80dee3375c026f74c6ffce88a942620588f915cfa1fae39baf1-Screenshot_2026-10-20_at_3.56.50_PM.png)

<br />

Once activated, Netacea will automatically apply the Monetize action to matching traffic and hand off to TollBit for licensing and content delivery.&#x20;

![](https://files.readme.io/cfd7ec50a1ef382da08da2396ecf8214ee975b6b0c3c89a4224aef2dd86ab7b3-Screenshot_2026-10-20_at_3.57.30_PM.png)

<br />

<br />
