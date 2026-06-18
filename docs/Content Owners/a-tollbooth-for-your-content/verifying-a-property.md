---
title: 'Step 1: Verify Property'
excerpt: Learn how to verify your property before enabling Analytics & Agent Site.
---
When you first log in to your org, you will be taken to the property overview page. If you don't have any properties configured, you will see this form that you can use to create your property. Create your property by giving it a name and inputting the domain name, i.e. `example.com` for your property.


<Image src="https://files.readme.io/d87c20bacea844167e1d608f6685203cbbce252086314ed6ba4070d624f34792-Screenshot_2026-02-24_at_5.28.32_PM.png" align="center" />


Once you've added your property, we will automatically fetch your sitemap through your `robots.txt` to determine your site's layout.

> 📘 Pro Tip
>
> We recommend publishers maintain a `robots.txt` file and a sitemap that is kept up to date. However, this is not required to onboard onto TollBit.

<br />

# Verifying a Property

Once you create a property, TollBit needs to verify your ownership of the domain and spin up your agent site. The records to be placed in your DNS registry will follow right after you have created your property.


<Image src="https://files.readme.io/d4f5b71769fe63ac71ae592e7247e2617c9856f132670e81d086ed7851d98bdd-Screenshot_2026-02-24_at_5.29.47_PM.png" align="center" />


##

### Step 1: Create TXT Record in your DNS registry

In your domain's DNS settings, create a TXT record for your top level domain (`example.com`, **not** `tollbit.example.com`. This might be a field called "name" or "host" when creating this record). Copy the TXT record value from the TollBit dashboard (it will look like `tollbit-domain-verification=...`) and paste it into the TXT you created. If you are unable to put the TXT record on your top level domain (if you already have a CNAME), you can put the TXT record on the `_tollbit-validation` subdomain. Looking at the full host, this would be `_tollbit-validation.example.com`.

### Step 2: Set Up Subdomain in your DNS registry

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

> 📘 Pro Tip
>
> Depending on your DNS provider, there may be a short period between saving your DNS updates and when the changes are visible to TollBit. This is normal and we give you the ability to continue with the onboarding process while you wait for these changes to propagate

<br />

## Bulk Updating DNS Records

##

This guide covers how to update TXT and NS records across multiple sites using the DNS management tools your publishers already use. Whether you're rolling out new SPF/DKIM/DMARC records, changing nameservers, or making platform-wide DNS changes, each registry handles bulk operations differently — use the links below to find the right workflow for your provider.


For any bulk update (10+ sites), we recommend using each provider's API or CLI tooling rather than their dashboard, which typically requires editing domains one at a time. Always back up existing records and lower your TTL before making changes.

**DNS Provider References**

- **GoDaddy** — Manage DNS records via the dashboard or REST API; <Anchor target="_blank" href="https://www.godaddy.com/help/manage-dns-records-680">DNS Management Help&#x20;</Anchor>| <Anchor target="_blank" href="https://developer.godaddy.com/doc/endpoint/domains#/">API Docs</Anchor>
- **Amazon Route 53&#x20;**— AWS's DNS service supports bulk changes via JSON change-batches applied through the AWS CLI or SDK. <Anchor target="_blank" href="https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/">Developer Guide</Anchor> | <Anchor target="_blank" href="https://docs.aws.amazon.com/cli/latest/reference/route53/">CLI Reference</Anchor>
- **Cloudflare** — Offers a REST API, Terraform provider, and the flarectl CLI for scripted bulk DNS record management. <Anchor target="_blank" href="https://developers.cloudflare.com/api/resources/dns/subresources/records/">DNS API Docs&#x20;</Anchor>| <Anchor target="_blank" href="https://registry.terraform.io/providers/cloudflare/cloudflare/latest/docs/resources/dns_record">Terraform Provider</Anchor>
- **Azure DNS** — Microsoft's cloud DNS integrates with Azure CLI and PowerShell for scripted bulk record management across zones. <Anchor target="_blank" href="https://learn.microsoft.com/en-us/azure/dns/">Azure DNS Docs</Anchor> | <Anchor target="_blank" href="https://learn.microsoft.com/en-us/cli/azure/network/dns">CLI Reference</Anchor>
- **Namecheap** — API access (with IP whitelisting) supports bulk changes via setHosts, which replaces all records for a domain at once. <Anchor target="_blank" href="https://www.namecheap.com/support/api/methods/domains-dns/set-hosts/">API Docs</Anchor> | <Anchor target="_blank" href="https://www.namecheap.com/support/knowledgebase/category/10/dns/">DNS Help</Anchor>
- **Google Domains / Squarespace** — Domain management moved to Squarespace in 2023; DNS records are managed per-domain through the Squarespace dashboard with no bulk API available.

<br />

## Whitelist TollBot user agent

##

Once your property is verified, TollBit recommends whitelisting the TollBot user agent. This allows the platform to run periodic reviews o understand site structure and content changes over time. The fetched site map can then be indexed for analytics and content delivery to LLMs for the fee that you choose.

Our user agent will be called "tollbot", stylized as "tollbot/1.0". Please whitelist this user agent. You can also find our IPs here: [https://tollbit.com/static-ips.txt](https://tollbit.com/static-ips.txt).

<br />

_Note that the syncs are setup initially once a site is onboarded, and then nightly afterwards to ensure we have an up to date structural understanding of the site. There may also be periodic requests to a small number of main pages or feeds to understand what changed._

<br />

<br />
