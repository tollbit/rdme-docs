---
title: Verification
excerpt: Learn how to verify your property ownership through DNS configuration.
---
# Verifying a Property

Once you create a property, we need to verify your ownership of the domain before we can continue with your TollBit integration. Once you have created a property and invited the necessary team members, you will see the property verification page below:

<Image align="center" src="https://files.readme.io/d4f5b71769fe63ac71ae592e7247e2617c9856f132670e81d086ed7851d98bdd-Screenshot_2026-02-24_at_5.29.47_PM.png" />

## DNS Configuration

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

<Callout icon="📘">
  Depending on your DNS provider, there may be a short period between saving your DNS updates and when the changes are visible to TollBit. This is normal and we give you the ability to continue with the onboarding process while you wait for these changes to propagate
</Callout>

## Next Steps

Once your property is verified, you can proceed to set up [integrations](integrations) and configure your [marketplace settings](marketplace).