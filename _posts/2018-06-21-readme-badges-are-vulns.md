---
layout: post
title:  "README badges are vulnerabilities"
date:   2018-06-22 12:40:00 -0000
categories: github badges vulns security
---

# Introduction

Badges are those cute little colourful rectangles that you see at the top of many READMEs. They often display things like the current version of the software, number of downloads, [code coverage](https://coveralls.io/), [results of vulnerability scans](https://snyk.io/docs/badges/), [build status](https://docs.travis-ci.com/user/status-images/), etc.

![Screenshot of a GitHub repo's README, with many badges]({{ "/img/badges.png" }})

# How do badges work?

Badges are simply images, nothing more. When you add a badge to your README, you add some code that looks like this:

Markdown:

```markdown
[![pypi](http://img.shields.io/pypi/v/pyhocon.png)](https://pypi.python.org/pypi/pyhocon)
```

which is rendered as HTML code that looks like:

```html
<a href="https://pypi.python.org/pypi/pyhocon" rel="nofollow">
  <img src="http://img.shields.io/pypi/v/pyhocon.png" alt="pypi">
</a>
```

(I know that's not exactly how it's rendered on GitHub. For example, GitHub proxies the images [through camo](https://github.com/atmos/camo). I've simplified the example for clarity.)

What happens when a user visits your README? Their browser sees these image tags and makes additional requests to fetch those images from the badge provider. Then the badge provider serves the images and the badges appear.

# What's the risk?

What happens when a badge provider:

* goes out of business and someone else buys the domain?
* decides they don't want to pay for the bandwidth anymore?
* inject ads into badges to pay for the bandwidth?
* sells tracking data about your users to pay for the bandwidth?
* gets hacked to inject malware into badges?

In all of these cases, your users could be exposed to "bad stuff". We've actually seen examples of similar scenarios in the past:

pypip.in went out of business and [all their badges broke](https://movermeyer.com/2018-03-08-building-bots-to-mend-badges/). Eventually a benevolent developer bought the domain and is now serving redirects to the [shields.io](https://shields.io/) badge service. The domain could have easily been bought by someone with malicious intents.

A photographer was [tired of paying bandwidth costs](#website-defacement) and swapped out a politician's banner image with a photoshopped image of the politician having sex. Any time you 'hotlink' to an image hosted by a third party, you are fundamentally trusting them not to swap out the contents.

CircleCI recently changed [their badges](https://circleci.com/docs/2.0/status-badges/) to include [an advertisement that their 1.0 API is being sunsetted](https://discuss.circleci.com/t/add-way-to-clear-1-0-is-sunsetting-on-status-badge/23085). While this case was well intentioned, it could have easily been something less charitable. To their credit they realised their mistake and reverted back to the regular badges.

There have been many instances of hackers [injecting malware](https://en.wikipedia.org/wiki/Malvertising#History) as part of XSS attacks on advertising or other images.

# I'm concerned... what should I do?

1. [Don't panic!](https://en.wikipedia.org/wiki/Phrases_from_The_Hitchhiker%27s_Guide_to_the_Galaxy#Don't_Panic) There's probably no need to be worried. 
2. Take a look at the badges that your projects are currently using and see who you are implicitly trusting today.
3. Make a conscious decision about whether the value provided by the badge outweighs the potential risks of allowing third-parties to host them.

In some cases, you might simply decide to drop the badge from your README.

In cases where the number of unique badge values is finite (ex. build passed or failed), you might be able to apply [Sub-Resource Integrity values](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) to the badge. This would prevent badge providers from returning anything other than the expected badges. However, I suspect that this would prove too brittle/infeasible in practice ([*let me know*](https://twitter.com/movermeyer)).

# Who can I trust?

This will depend on your project's needs.

In my mind, the large badge providers are more trustworthy, as they usually have a source of revenue that relies on continued customer/user trust. 

[shields.io](https://shields.io/) is probably the largest badge provider today. It's an open-source community project that doesn't do any tracking of user requests. They are [funded](https://github.com/badges/shields/issues/1377#issuecomment-352842533) [through donations](https://opencollective.com/shields). Personally, I consider them to be trustworthy for my projects. Of course, by being the largest, they also have the largest potential value to hackers.

(**DISCLAIMER:** I have [previously donated](https://opencollective.com/movermeyer) to shields.io)

If you simply don't like the idea of trusting any third party badge providers, you can always host your own instance of [the open-source shields.io server](https://github.com/badges/shields/blob/master/doc/self-hosting.md).

# Conclusions

Badges are a security vulnerability, and maintainers of projects should frame their thinking around them in that way. You shouldn't necessarily be worried, but you should definitely be aware of the risks they pose to your project.

The open-source community at large is getting more aware of security risks throughout the ecosystem, [from dependencies](https://hackernoon.com/part-2-how-to-stop-me-harvesting-credit-card-numbers-and-passwords-from-your-site-844f739659b9) to [reproducible builds](https://reproducible-builds.org/). Thinking about README badge providers as a calculated risk vulnerability is just another part of the overall hardening process.

---

* <a name="website-defacement"></a> *If you remember more about that incident with the photographer and the politician, I'd love to be able to link to the article. Here are [the details as I remember them](https://www.reddit.com/r/tipofmytongue/comments/8ss8r4/tomtnews_italian_politician_gay_sex_website/).*