# Transmitly.ChannelProvider.Mailgun

`Transmitly.ChannelProvider.Mailgun` is the convenience package for sending email with [Transmitly](https://github.com/transmitly/transmitly) through [Mailgun](https://www.mailgun.com/).

This package is the one most applications should install. It wires together:

- `Transmitly.ChannelProvider.Mailgun.Configuration`
- `Transmitly.ChannelProvider.Mailgun.Api`

In practice, that means one package gives you:

- `AddMailgunSupport(...)` for registering Mailgun with `CommunicationsClientBuilder`
- Mailgun email dispatch support for the `Email` channel
- Mailgun-specific email configuration via `email.Mailgun()`
- Mailgun delivery report adaptation and Mailgun-specific delivery report properties via `report.Mailgun()`

## Install

```shell
dotnet add package Transmitly.ChannelProvider.Mailgun
```

## Quick Start

```csharp
using Transmitly;

ICommunicationsClient communicationsClient = new CommunicationsClientBuilder()
	.AddMailgunSupport(options =>
	{
		options.ApiKey = "key-your-mailgun-api-key";
		options.SendingDomain = "mg.example.com";

		// Use this for EU-region Mailgun accounts when needed.
		// options.ApiHost = "https://api.eu.mailgun.net";
	})
	.AddPipeline("WelcomeEmail", pipeline =>
	{
		pipeline.AddEmail("welcome@example.com".AsIdentityAddress("Example App"), email =>
		{
			email.Subject.AddStringTemplate("Welcome, {{firstName}}!");
			email.HtmlBody.AddStringTemplate("<strong>Welcome</strong> {{firstName}}!");
			email.TextBody.AddStringTemplate("Welcome {{firstName}}!");
		});
	})
	.BuildClient();

var result = await communicationsClient.DispatchAsync(
	"WelcomeEmail",
	"customer@example.com".AsIdentityAddress("Casey"),
	new { firstName = "Casey" });
```

## Required Options

At minimum, configure:

- `ApiKey`: your Mailgun API key
- `SendingDomain`: the Mailgun sending domain, such as `mg.example.com`

Additional options available on `MailgunOptions`:

- `ApiHost`: defaults to `https://api.mailgun.net`
- `ApiVersion`: defaults to `v3`
- `WebProxy`: optional `WebProxy` for outbound HTTP requests

## Mailgun-Specific Email Features

This package registers the Mailgun extended properties adapter, so you can configure provider-specific settings directly on an email channel:

```csharp
using System.Collections.Generic;
using Transmitly;

ICommunicationsClient communicationsClient = new CommunicationsClientBuilder()
	.AddMailgunSupport(options =>
	{
		options.ApiKey = "key-your-mailgun-api-key";
		options.SendingDomain = "mg.example.com";
	})
	.AddPipeline("ReceiptEmail", pipeline =>
	{
		pipeline.AddEmail("billing@example.com".AsIdentityAddress("Example Billing"), email =>
		{
			email.Subject.AddStringTemplate("Your receipt");
			email.HtmlBody.AddStringTemplate("<p>Thanks for your order.</p>");
			email.TextBody.AddStringTemplate("Thanks for your order.");

			email.Mailgun().Template = "receipt-template";
			email.Mailgun().TemplateVersion = "v1";
			email.Mailgun().Tags = ["receipt", "transactional"];
			email.Mailgun().TrackingOpens = true;
			email.Mailgun().TrackingClicks = true;
			email.Mailgun().Properties = new Dictionary<string, string>
			{
				["v:customer-id"] = "12345"
			};
		});
	})
	.BuildClient();
```

Provider-specific features exposed by `email.Mailgun()` include Mailgun template selection, tags, tracking settings, DKIM settings, AMP HTML, TLS requirements, sending IP selection, and custom Mailgun message properties.

## Delivery Reports

`AddMailgunSupport(...)` also registers the Mailgun delivery report adaptor. When you use the Transmitly MVC integrations to receive provider webhooks, Mailgun events can be normalized into Transmitly `DeliveryReport` instances and still expose Mailgun-specific details:

```csharp
.AddDeliveryReportHandler(report =>
{
	var mailgun = report.Mailgun().Email;

	logger.LogInformation(
		"Mailgun event={Event}, messageId={MessageId}, recipient={Recipient}",
		mailgun.Event,
		mailgun.MessageId,
		mailgun.Recipient);

	return Task.CompletedTask;
})
```

Mailgun-specific delivery report data exposed through `report.Mailgun().Email` includes values such as the Mailgun event name, recipient, tags, status details, message metadata, envelope details, and webhook signature fields.

## Multiple Mailgun Accounts

If you need to register more than one Mailgun account, pass the optional `providerId` parameter to `AddMailgunSupport(...)` to create distinct provider registrations.

```csharp
new CommunicationsClientBuilder()
	.AddMailgunSupport(options =>
	{
		options.ApiKey = "key-primary";
		options.SendingDomain = "mg.example.com";
	}, providerId: "primary")
	.AddMailgunSupport(options =>
	{
		options.ApiKey = "key-marketing";
		options.SendingDomain = "mg.marketing.example.com";
	}, providerId: "marketing");
```

See the main [Transmitly](https://github.com/transmitly/transmitly) project for broader documentation on pipelines, channels, template engines, dependency injection, and delivery report handling.

---
_Copyright (c) Code Impressions, LLC. This open-source project is sponsored and maintained by Code Impressions and is licensed under the [Apache License, Version 2.0](http://apache.org/licenses/LICENSE-2.0.html)._
