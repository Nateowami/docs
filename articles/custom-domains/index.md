---
title: Custom Domains
description: How to map custom domains
toc: true
topics:
  - custom-domains
contentType: how-to
useCase: customize-domains
---
# Custom Domains

Auth0 allows you to map the domain for your tenant to a custom domain of your choosing. This allows you to maintain a consistent experience for your users by keeping them on your domain instead of redirecting or using Auth0's domain.

For example, if your Auth0 domain is **northwind.auth0.com**, you can have your users to see, use, and remain on **login.northwind.com**.

We recommend that you use custom domains with Universal Login for the most seamless and secure experience for your users. Check the [Universal Login documentation](/hosted-pages/login) to see if your plan and use case support custom domains. 

## Prerequisites

* This feature is not available for free plans. To configure a custom domain you have to [upgrade your account to any paid plan](${manage_url}/#/tenant/billing/subscription)
* You must register and own the domain name to which you are mapping your Auth0 domain

## Features supporting use of custom domains

Currently, the following Auth0 features and flows support the use of custom domains:

* OAuth 2.0/OIDC-Compliant flows (those using the [`/authorize`](/api/authentication#authorize-application) and [`/oauth/token`](/api/authentication#get-token) endpoints)
* Guardian (MFA Widget Version 1.3.3/Guardian.js Version 1.3.0 or later)
* Emails (the links included in the emails will use your custom domain)
* Database and social connections
* Lock 11 with cross-origin authentication
* <dfn data-key="passwordless">Passwordless</dfn> connections with Universal Login (The email link will be sent using the custom domain if the option is enabled in **Tenant Settings > Custom Domains**)
* G Suite connections
* <dfn data-key="security-assertion-markup-language">SAML</dfn> connections and applications
* WS-Fed clients (Auth0 as IDP using WS-Fed Add-on)
* Azure AD connections
* ADFS connections
* AD/LDAP connections

:::warning
Features not in the list are **not supported** by Auth0 with custom domains.
:::

::: panel Token issuance
Auth0 issues tokens with the **iss** claim of whichever domain you used with the request. For example, if you used **https://northwind.auth0.com/authorize...** to obtain an Access Token, the **iss** claim of the token you receive will be **https://northwind.auth0.com/**. If you used your custom domain **https://login.northwind.com/authorize...**, the **iss** claim value will be **https://login.northwind.com/**. 

If you get an Access Token for the [Management API](/api/management/v2) using an authorization flow with your custom domain, you **must** call the Management API using the custom domain (your token will be considered invalid otherwise).
:::

## Certificate management

Auth0 offers two certificate management options.

* **Auth0-Managed Certificates**: Auth0-managed certificates are those where Auth0 will manage the creation and renewal of the certificates for your custom domain. This is the simplest custom domains deployment option, and the scope of this document

* **Self-Managed Certificates**: [Self-managed certificates](/custom-domains/self-managed-certificates) are available for enterprise customers only. Choosing this option means that you are responsible for managing your SSL/TLS certificates and configuring a reverse proxy to handle SSL termination and forwarding requests to Auth0.

## How to configure custom domains

Setting up your custom domain using Auth0-managed certificates requires you to do the following steps:

1. Provide your domain name to Auth0
1. Verify ownership
1. Complete feature-specific setup

### Step 1: Provide your domain name to Auth0

Log in to the Dashboard and go to [Tenant Settings](${manage_url}/#/tenant). Click over to the **Custom Domains** tab.

![](/media/articles/custom-domains/custom-domains.png)

Enter your custom domain in the provided box and select **Auth0-managed certificates**. Click **Add Domain**.

### Step 2: Verify ownership

Before you can use this domain, you'll need to verify that you own your domain. To do this, you need to add the CNAME verification record listed in the Dashboard to your domain's DNS record.

![](/media/articles/custom-domains/auth0-managed.png)

When you've done so, click **Verify** to proceed.

::: note
It may take a few minutes before Auth0 is able to verify your CNAME record, depending on your DNS settings.
:::

::: panel TL;DR;
Here's how to add the CNAME verification record to your domain's DNS record. The steps specified may vary by domain host provider, but generally speaking, you will need to:

1. Log in to your domain management service (such as GoDaddy or Google Domains)

1. Create a new record:

	* For the record type, indicate **CNAME**
	* For the **Name** field, enter your custom domain name (such as `login.northwind.com`)
	* Leave the **Time to Live (TTL)** field set to the default value
	* In the **Value** field, paste in the CNAME value provided by the Auth0 Dashboard

When done, save your record.
:::

If Auth0 was able to verify your domain name, you'll see a confirmation window. 

This means the verification process is complete and within 1 to 2 minutes, your custom domain should be ready to use.

![](/media/articles/custom-domains/domain-verification.png)

::: warning
If you are unable to complete the verification process within three days, you'll need to start over.
:::

::: warning
Once added, the CNAME record must be present at all times to avoid issues during certificate renewal.
:::

### Step 3: Additional configuration steps

There may be additional steps you must complete depending on which Auth0 features you are using. See the [Additional Configuration for Custom Domains](/custom-domains/additional-configuration) document for more information.

## FAQ

1. **If I use a custom domain, will I still be able to use my ${account.namespace} domain to access Auth0?**
  
Yes, you will be able to use either the default `${account.namespace}` or your custom domain. There are, however, a few exceptions:

- If you are using embedded Lock or an SDK, the configuration is pre-defined as using either your custom domain or the `${account.namespace}` domain, so you have to use one or the other
- If you start a session in `${account.namespace}`, and go to `custom-domain.com`, the user will have to login again

2. **How many custom domains can I use per tenant?**

Currently, each tenant supports one custom domain.

3. **Can you provide me a static list of IP addresses for my custom domain so I can whitelist them?**

We cannot provide a static list of IP addresses as they are subject to change. Our recommendation is to whitelist your custom domain instead.

## Troubleshooting

If you are seeing errors, take a look at this video on common issues with Custom Domains, or refer to the below sections for troubleshooting steps for specific scenarios.

<iframe width="560" height="315" src="https://www.youtube.com/embed/f6fkOkS1RFA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Custom domain is still pending verification

If you continue to see this error in the Dashboard, make sure that the CNAME record is properly configured in your domain management service.

You can confirm the configuration of your CNAME record using:

* A tool like [Mxtoolbox](https://mxtoolbox.com/CNAMELookup.aspx) or [Google](https://dns.google.com)
* The `dig` command in your terminal

Please remember that it can take up to 48 hours for the DNS to be propagated.

### CNAME flattening

Cloudflare has a service called CNAME Flattening. During the verification process, turn off the CNAME flattening process until the domain verification steps are complete to prevent IP address confusion.

### "You should not be hitting this endpoint"
If you see this error when configuring a custom domain, you must perform [additional configuration](/custom-domains/additional-configuration), which varies depending on your setup.

### "Service not found"

If your application issues an `/authorize` request with `audience=https://login.northwind.com/userinfo`, the server will return a `Service not found: https://login.northwind.com/userinfo` error. This is because even if you set a custom domain the API identifier for the `/userinfo` endpoint remains `https://{YOUR_ORIGINAL_AUTH0_DOMAIN}/userinfo`. 

To fix this your app should instead use `audience=https://{YOUR_ORIGINAL_AUTH0_DOMAIN}/userinfo`. You can also remove this `audience=[...]/userinfo` parameter altogether if your application is flagged as **OIDC-Conformant** in the **OAuth2** tab of the application's **Advanced Settings**.

### Errors related to Internet Explorer

If you are using Internet Explorer, you may see any of the following error messages:

* "No verifier returned from client"
* "Origin header required"
* "Failed cross origin authentication"

#### Why you see these errors

When both the Auth0 domain and the app domain are in the same trusted or local intranet zone, Internet Explorer does *not* treat the request as a cross-domain request and therefore does not send the cross-origins header.

If you see any of these errors and you are using Embedded Login, you can move one of the sites out of the trusted or local intranet zone. To do this:

1. Go to Internet Options > Security. 
2. Select the **Local Intranet Zone** tab and go to Sites > Advanced. Add your domain.
3. Return to the **Security** tab, and make sure the proper zone has been selected.
4. Click **Custom Level** and look for **Access data sources across domains** under the **Miscellaneous** section. Check the radio button next to **Enable.**.

Alternatively, you can remove reliance on cross-origin authentication by implementing [Universal Login](/hosted-pages/login).
