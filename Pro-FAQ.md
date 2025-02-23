Karafka Pro is an enhanced version of the Karafka framework, adding more functionalities and providing additional customer support options.

1. [Is there a trial version?](#is-there-a-trial-version)
2. [What is the license?](#what-is-the-license)
3. [How does Pro licensing work?](#how-does-pro-licensing-work)
4. [Do I require to change source of the package?](#do-i-require-to-change-source-of-the-package)
5. [What happens if my subscription lapses?](#what-happens-if-my-subscription-lapses)
6. [Do I need to replace the license for my running processes?](#do-i-need-to-replace-the-license-for-my-running-processes)
7. [How do I buy Karafka Pro?](#how-do-i-buy-karafka-pro)
8. [Can I distribute Karafka Pro to my customers?](#can-i-distribute-karafka-pro-to-my-customers)
9. [Can you transfer a license?](#can-you-transfer-a-license)
10. [Do I have to share the credentials with all of my developers?](#do-i-have-to-share-the-credentials-with-all-of-my-developers)
11. [Can I get a refund?](#can-i-get-a-refund)
12. [Can I accidentally use Pro because it is in the same repository?](#can-i-accidentally-use-pro-because-it-is-in-the-same-repository)
13. [Why do I see a "Bad username or password" message when trying to bundle install?](#why-do-i-see-a-bad-username-or-password-message-when-trying-to-bundle-install)
14. [Where can I find my license credentials page URL?](#where-can-i-find-my-license-credentials-page-url)
15. [Do you require any personal information, financial data, confidential/sensitive data, government data, etc.?](#do-you-require-any-personal-information-financial-data-confidentialsensitive-data-government-data-etc)
16. [How do you ensure security of your gem server?](#how-do-you-ensure-the-security-of-your-gem-server)
17. [Does Karafka Pro stores, processes or transmits Personal Health Information?](#does-karafka-pro-stores-processes-or-transmits-personal-health-information)
18. [Does Karafka Pro stores, processes or transmits Personally Identifiable Information?](#does-karafka-pro-stores-processes-or-transmits-personally-identifiable-information)
19. [Will Karafka store, process, or transmit company confidential information and data?](#will-karafka-store-process-or-transmit-company-confidential-information-and-data)
20. [How do you store, process, or transmit Payment Cardholder Information?](#how-do-you-store-process-or-transmit-payment-cardholder-information)
21. [What specific PHI, PII, PCI data fields, or company confidential information and data do you collect?](#what-specific-phi-pii-pci-data-fields-or-company-confidential-information-and-data-do-you-collect)
22. [Where is the hosting infrastructure located?](#where-is-the-hosting-infrastructure-located)
23. [Can I use Karafka Pro with an offline license without using the Karafka gem server?](#can-i-use-karafka-pro-with-an-offline-license-without-using-the-karafka-gem-server)
24. [Ethics, Privacy, and Information Usage](#ethics-privacy-and-information-usage)
25. [Contact Info](#contact-info)

## Is there a trial version?

Yes. For free, you can obtain temporary credentials from our [website](https://karafka.io/#become-pro).

Those credentials will be valid for 14 days for every environment.

The trial license does **not** grant you our Pro commercial support.

## What is the license?

See [LICENSE-COMM](https://github.com/karafka/karafka/blob/master/LICENSE-COMM) in the root of the Karafka repo.

## How does Pro licensing work?

Every organization running Karafka Pro on its servers must purchase a subscription. There's no limit to the number of servers or environments used by that organization. Your subscription will automatically renew every year.

## Do I require to change source of the package?

**No**. All Karafka Pro code is stored in the same package and only included and used when a valid license gem is present.

## What happens if my subscription lapses?

If we cannot charge your card, we will email you and try three more times over a week. If it still fails, your subscription will be canceled.

You'll lose access to the gem server and priority support, and Karafka Pro won't work anymore.

## Do I need to replace the license for my running processes?

No. The production environment that is already started will not be affected (until the next deployment).

## How do I buy Karafka Pro?

Follow the instructions on our [website](https://karafka.io/#become-pro).

## Can I distribute Karafka Pro to my customers?

This is a common requirement for "on-site installs" or "appliances" sold to large corporations.

The standard license is only appropriate for SaaS usage as it does **not** allow distribution. Karafka Pro has an Appliance license option which does allow you to distribute it. The Appliance license is $2,995/yr. It allows you to distribute the Pro gem as part of your application and each of your customers to run Karafka Pro as part of your application only. Email contact@karafka.io to purchase.

## Can you transfer a license?

Licenses are **not** transferrable to another company. It is strongly recommended that you buy the license using a group email address so the license is not attached to any one employee's email address.

## Do I have to share the credentials with all of my developers?

In general, **yes**. The credentials are required to download the gems and your developers will need the gems to use the commercial features.

## Can I get a refund?

**No**. We offer a 14 days trial during which you can check out Karafka Pro capabilities.

## Can I accidentally use Pro because it is in the same repository?

**No**. The Pro code is **never** loaded unless a valid `karafka-license` is detected.

## Why do I see a "Bad username or password" message when trying to bundle install?

If you are seeing the following error when trying to `bundle install`:

```
Fetching source index from https://gems.karafka.io/

Bad username or password for https://LOGIN@gems.karafka.io/.
Please double-check your credentials and correct them.
```

1. Check the account email's Spam/Junk folder for any billing or payment emails. You would need to purchase a new subscription if your subscription was canceled.

2. Double-check your login and password to the gem server.

3. Upgrade Bundler. Versions before 2.3.x are buggy with the gem server.

## Where can I find my license credentials page URL?

You can find it in the email you received from us when you requested the Pro license via our gems UI.

## Do you require any personal information, financial data, confidential/sensitive data, government data, etc.?

No. We only collect enough customer information to fill out a standard invoice for billing purposes, and customer information is never shared or sold to anyone.

Karafka license gem contains **only** the organization name.

## How do you ensure the security of your gem server?

Please read our [Security statement](Pro-Security) for details on this manner.

## Does Karafka Pro stores, processes or transmits Personal Health Information?

**No**. Karafka and Karafka Pro do not store, process or transmit any information to me. Never.

## Does Karafka Pro stores, processes or transmits Personally Identifiable Information?

**No**. Karafka and Karafka Pro do not store, process or transmit any information to me. Never.

## Will Karafka store, process, or transmit company confidential information and data?

**No**., it will not store, process, or transmit company confidential information and data.

## How do you store, process, or transmit Payment Cardholder Information?

We do not hold your credit card information. All subscription management is done via Stripe.

## What specific PHI, PII, PCI data fields, or company confidential information and data do you collect?

**No**. Karafka and Karafka Pro do not store, process or transmit any information to me. Never.

## Where is the hosting infrastructure located?

Karafka Pro gem license server is located in Hetzner, Germany, Frankfurt. Karafka and Karafka Pro are served directly from [RubyGems](https://rubygems.org).

## Can I use Karafka Pro with an offline license without using the Karafka gem server?

We understand that some companies have strict policies regarding their open-source supply chain, and we are happy to provide a solution that meets those needs.

Karafka Pro can be used with an embedded/offline license without relying on our gem server. It is important to note that this mode of operation requires an additional agreement, and there is an additional cost of USD 1000 yearly. Upon agreement, we will provide the license gem with installation instructions, so you can start using Karafka Pro with ease without reliance on any third-party sources.

## Ethics, Privacy, and Information Usage

We only collect enough customer information to fill out a standard invoice for billing purposes. Customer information is never shared or sold to anyone.

The Karafka software runs on your servers. Karafka never has access to any private user data.

## Contact Info

```
Maciej Mensfeld, Karafka.io
Sikorskiego 31/12
34-400 Nowy Targ
Poland

All billing/support inquiries: contact@karafka.io
NIP (VAT-ID): PL 735 261 5885
```
