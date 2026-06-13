---
title: Content Controls & Governance
excerpt: >-
  Control what data to include or exclude when developers request content from
  your website through TollBit.
deprecated: false
hidden: false
metadata:
  robots: index
---
<br />

<Image align="center" src="https://files.readme.io/a8b8033336babf75aa0f824b217f1d636408b45bc1ce23699bd1e82b5f6c01a0-Screenshot_2026-02-26_at_10.10.44_AM.png" />

### Element Filters

Element filters help exclude certain types of content from the HTML of your website, such as removing all images, links or embedded content from being returned to agentic consumers. Note that in order for this feature to work properly, the content to be excluded needs to be written in well formatted HTML. For example, we won't be able to exclude a hyperlink if it's not within an `<a>` tag.

<Image align="center" src="https://files.readme.io/196d471c4942aba3917ab7b6f4bce341cd0fd77983c58c3d0936908fcfc3cbe4-Screenshot_2026-02-26_at_10.11.26_AM.png" />

For more advanced use cases you can exclude all elements that are styled with a specific CSS class.

<Image align="center" src="https://files.readme.io/e05e5e1744393a3b9ea7bf622f0e687e9a3bfe29245cbfd9a739973aca1fe721-Screenshot_2026-02-26_at_10.12.14_AM.png" />

### Article Filters

Article filters allow you to programmatically exclude entire HTML pages from AI access. An article filter can be configured to match a specific HTML tag with specific attributes and attribute values. Any page that contains the tag that matches will be excluded from agentic access. This feature is intended to be used to exclude content like syndicated articles.

<Image align="center" src="https://files.readme.io/4386ee3fe1833c5930518aeb1854c975f86193b0c07e630148e00e8a5aa7046c-Screenshot_2026-02-26_at_10.13.11_AM.png" />

If you already have a `<meta name='robots' content='noindex,nofollow'>` tag (or similar) to exclude pages from traditional web crawling, configuring an article filter to match that tag will also exclude pages containing that tag from AI access.

If you already have a `<meta name='robots' content='noindex,nofollow'>` tag (or similar) to exclude pages from traditional web crawling, configuring an article filter to match that tag will also exclude pages containing that tag from AI access.

<Image align="center" src="https://files.readme.io/7190c766043b6666cc76bec976a9d9bdc56e61ea3f48a8daa24022eab90058b1-Screenshot_2026-02-26_at_10.14.29_AM.png" />

<br />