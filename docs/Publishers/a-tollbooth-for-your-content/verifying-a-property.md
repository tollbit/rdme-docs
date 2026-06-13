---
title: 'Step 1: Verify Property'
excerpt: Learn how to verify your property ownership through DNS configuration.
---
To get all the benefits of TollBit, including our analytics and agent site set up, the first and most important step is to add and set up your properties.

# Adding a Property

When you first log in to your org, you will be taken to the property overview page. If you don't have any properties configured, you will see this form that you can use to create your property. Create your property by giving it a name and inputting the domain name, i.e. `example.com` for your property.


<Image src="https://files.readme.io/d87c20bacea844167e1d608f6685203cbbce252086314ed6ba4070d624f34792-Screenshot_2026-02-24_at_5.28.32_PM.png" align="center" />


Once you've added your property, we will automatically fetch your sitemap through your `robots.txt` to determine your site's layout.

> 📘
>
> We recommend publishers maintain a `robots.txt` file and a sitemap that is kept up to date. However, this is not required to onboard onto TollBit.

<br />

# Verifying a Property

Once you create a property, we need to verify your ownership of the domain before we can continue with your TollBit integration. Once you have created a property and invited the necessary team members, you will see the property verification page below:


<Image src="https://files.readme.io/d4f5b71769fe63ac71ae592e7247e2617c9856f132670e81d086ed7851d98bdd-Screenshot_2026-02-24_at_5.29.47_PM.png" align="center" />


## DNS Configuration

<br />

### Step 1: Create TXT Record

In your domain's DNS settings, create a TXT record for your top level domain (`example.com`, **not** `tollbit.example.com`. This might be a field called "name" or "host" when creating this record). Copy the TXT record value from the TollBit dashboard (it will look like `tollbit-domain-verification=...`) and paste it into the TXT you created. If you are unable to put the TXT record on your top level domain (if you already have a CNAME), you can put the TXT record on the `_tollbit-validation` subdomain. Looking at the full host, this would be `_tollbit-validation.example.com`.

### Step 2: Set Up Subdomain

As part of verification, we will also set up the `tollbit` subdomain in your registrar. This allows users to easily access the content mirrored on your main website. This will not affect your main website's SEO, load times, etc.

Navigate to your DNS provider and create a new NS record for the subdomain. If your main website were `www.example.com` or `example.com`, the subdomain must be `tollbit.example.com`. Point the NS records at the following domains:

```
ns1.edge.tollbit.com
```

```
ns2.edge.tollbit.com
```

```
ns3.edge.tollbit.com
```

```
ns4.edge.tollbit.com
```

> 📘
>
> Depending on your DNS provider, there may be a short period between saving your DNS updates and when the changes are visible to TollBit. This is normal and we give you the ability to continue with the onboarding process while you wait for these changes to propagate

<br />

## Whitelist TollBot user agent

##

We run periodic review on the main pages of TollBit publisher sites to understand site structure and content changes over time. The fetched site map can then be seen in the TollBit dashboard (under Summarization License Options > Directory).

Note that the syncs are setup initially once a site is onboarded, and then nightly afterwards to ensure we have an up to date structural understanding of the site. There may also be periodic requests to a small number of main pages or feeds to understand what changed.

Our user agent will be called "tollbot", stylized as "tollbot/1.0". Please whitelist this user agent. You can also find our IPs here: [https://tollbit.com/static-ips.txt](https://tollbit.com/static-ips.txt).

<br />
