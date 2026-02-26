---
title: Customization for Rates
deprecated: false
hidden: false
metadata:
  robots: index
---
<br />

### Bot Rates

These rates allow you to set special rates for any specific bots that access your platform, and will override all other rates. You should set this type of rate if you have struck a licensing deal with a company that employs a particular user agent, and want to give them special rates to access your content (usually 0).

### Page Rates

These rates allow you to set a rate for a specific page on your website. If you have any page that you know gets high bot traffic (i.e. sports or election results), or if you have a very high quality piece of original reporting, you can set a special rate for that page. This will override all other rates except bot rates.

### Keyword Rates

These rates allow you to set a price for pages that may contain a particular keyword. If you know that there are some high profile sporting events coming up, you may want to set a higher price for pages that mention `football` or `basketball`. This rate is still in beta.

### Directory Rates

These rates let you set a flat fee for all the content within a page directory of your site. For a quick way to instantly price your content, you can set a price for your top level directory, and this will automatically apply to all pages. You can drill down into further subdirectories and set pricing there, and it will override any price in a higher directory. For example, you can set a base price of $0.001 at the root level, and then set a price of $0.005 for the `/sports` directory. Everything under `/sports` will now be $0.005 while something under `/cooking` will still be $0.001.
