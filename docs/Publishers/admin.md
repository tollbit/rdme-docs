---
title: Admin
excerpt: Manage your organization, invite members, and configure payment settings.
---

# Manage your Organization

If you are an admin for your organization, you will be able to invite other members to the TollBit dashboard. To get started, navigate to the [Settings](https://app.tollbit.com/settings) page.

## Invite Members

You can invite new members to your organization and assign them roles when they join.

<Image src="https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/invite-members.png" alt="Invite Members" />

## Manage Members

You can manage your existing members, change their roles, or remove them from your organization.

<Image src="https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/manage-members.png" alt="Manage Members" />

## Add payment

In order to receive payment for the access to your content, you must add your payment information to your account. We partner with [Stripe](https://stripe.com) to process payments. You will be able to add either your bank account or a debit card where we will deposit payments to. Payouts will occur in the first couple days of the month, and will be for all transactions that took place in the previous month. More specifically, this will be from `00:00:00.000 UTC` of the first day of the previous month, to `23:59:59.999 UTC` of the last day of the previous month.

<Callout icon="ℹ️" theme="info">
  Only admins in your organization will be able to view and modify payment information. Read the [Organization](/docs/publishers/admin) section to see how to manage your org.
</Callout>

You can only add one payment method per organization. This is fine for most orgs, but for those that prefer to keep accounting separate for subsets of their properties, they can create and manage multiple orgs, each with its own set of properties, and set payment information separate for each org.

