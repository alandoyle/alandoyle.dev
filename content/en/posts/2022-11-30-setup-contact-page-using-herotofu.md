---
title: "Setup Contact Page Using HeroTofu"
description: ""
author:
email:
date: 2022-11-30T15:19:07Z
publishDate: 2022-11-30T15:19:07Z
images: []
draft: false
tags: []
cover:
    image: "/images/blog/markus-winkler-GTJNxRG4QJw-unsplash-cropped.jpg"
    alt: "Markus Winkler"
    relative: false
---

Having tried a number of different methods of receiving details via a Contact form I've settled on using [HeroTofu](https://herotofu.com)

I am currently using their [Free tier](https://herotofu.com/pricing) which limits the number of submissions to 100 per month (plenty for a tiny site like this) but actually includes a couple of really neat features.

1. Built-in Captcha to reduce spam or you can add your own Recaptcha secret key if you'd prefer to use Recaptcha instead. This can also be setup to only trigger on suspicious submissions, all submissions or never!

2. A custom "Thank you" page URL can be configures thus making the whole process seamless and keeps users on your site after they submitted so Contact info.

![HeroTofu](/images/blog/herotofu-logo.png)

# Setup

Setting up a new HeroTofu account and creating a Contact Form is really simple.

## Create Account

Head over to the [HeroTofu](https://app.herotofu.com/signup) website and create an account. You'll get 14 free days of the free trial which will become a free forever plan when it expires if you don't pay for a subscription. For the sole purpose of my contact form, this is more than enough.

![HeroTofu Signup](/images/blog/herotofu-signup.jpg)

Your HeroTofu account will be **On-Hold** until you confirm your email address.

## Create New Form

Login and select **Forms** from the lefthand panel, then click the **+ New Form** button.

![HeroTofu Forms Page](/images/blog/herotofu-forms-page.jpg)

The form is comprised of 3 sections.

### Setup

Fill in the **Form Name** and scroll down to the 2nd section.

![HeroTofu Setup](/images/blog/herotofu-form-setup.jpg)

### Lead Routing

Here you should fill in a URL on your website which HeroTofu will redirect to when a Contact Form has been submitted.

Also fill in an email address to receive notifications when a new Contact Form submission has been received.

![HeroTofu Lead Routing](/images/blog/herotofu-form-lead-routing.jpg)

### Additional Flags

Here you can provide a *Recaptcha secret key* or confiure the built-in Captcha system.

![HeroTofu Additional Flags](/images/blog/herotofu-form-additional-flags.jpg)

Click **Save** to create the form.

## Configuring [Cleite](https://github.com/alandoyle/hugo-cleite-theme) theme

Going back to the **Forms** yu will see a new Form exists.

![HeroTofu New Form](/images/blog/herotofu-new-form.jpg)

Edit the **config.toml** and use the URL displayed.

```toml
[params.form] # Contact form
    netlify = false
    action = "https://public.herotofu.com/v1/06bc0820-7236-11ed-90d2-f5e5b36b68af"
    method = "POST"
    inputNameName = "name"
    inputNameLabel = "Name"
    inputNamePlaceholder = "Your name"
    inputEmailName = "email"
    inputEmailLabel = "Email"
    inputEmailPlaceholder = "Your email"
    inputMsgName = "message"
    inputMsgLabel = "Write something"
    inputMsgLength = 750
    inputSubmitValue = "Send"
```

# Conclusion

That's it. An incredibly simple process which took less than 5 minutes to implement. If your Hugo theme doesn't include a built-in Contact Form like [Cleite](https://github.com/alandoyle/hugo-cleite-theme) then HeroTofu have some [additional information](https://herotofu.com/solutions/guides/hugo-contact-form) available on their website.