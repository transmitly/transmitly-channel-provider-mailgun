# Transmitly.ChannelProvider.Mailgun

`Transmitly.ChannelProvider.Mailgun` is the convenience package for sending email with [Transmitly](https://github.com/transmitly/transmitly) through [Mailgun](https://www.mailgun.com/).

This is the package most applications should install. It wires together:

- `Transmitly.ChannelProvider.Mailgun.Configuration`
- `Transmitly.ChannelProvider.Mailgun.Api`

Supported channels:

- `Email`

## Install

```shell
dotnet add package Transmitly.ChannelProvider.Mailgun
```

## Quick Start

```csharp
using Transmitly;

ICommunicationsClient client = new CommunicationsClientBuilder()
	.AddMailgunSupport(options =>
	{
		options.ApiKey = "key-your-mailgun-api-key";
		options.SendingDomain = "mg.example.com";
	})
	.AddPipeline("welcome-email", pipeline =>
	{
		pipeline.AddEmail("welcome@example.com".AsIdentityAddress("Example App"), email =>
		{
			email.Subject.AddStringTemplate("Welcome to Example App");
			email.HtmlBody.AddStringTemplate("<strong>Welcome</strong> to Example App.");
			email.TextBody.AddStringTemplate("Welcome to Example App.");
		});
	})
	.BuildClient();

var result = await client.DispatchAsync(
	"welcome-email",
	"customer@example.com".AsIdentityAddress("Customer"),
	new { });
```

## Configuration

`AddMailgunSupport(options => ...)` accepts `MailgunOptions`.

Common settings:

- `ApiKey`: your Mailgun API key.
- `SendingDomain`: the Mailgun sending domain, such as `mg.example.com`.
- `ApiHost`: defaults to `https://api.mailgun.net`.
- `ApiVersion`: defaults to `v3`.
- `WebProxy`: optional outbound proxy.

## Mailgun-Specific Email Features

This package registers Mailgun email extensions through `email.Mailgun()`.

Common provider-specific settings include:

- stored Mailgun templates via `Template` and `TemplateVersion`
- AMP email via `AmpHtml`
- tags and custom properties
- DKIM and TLS options
- Mailgun tracking flags

Example:

```csharp
using System.Collections.Generic;
using Transmitly;

pipeline.AddEmail("welcome@example.com".AsIdentityAddress("Example App"), email =>
{
	email.Subject.AddStringTemplate("Welcome to Example App");
	email.TextBody.AddStringTemplate("Welcome to Example App.");

	email.Mailgun().Tags = new[] { "welcome", "transactional" };
	email.Mailgun().Tracking = true;
	email.Mailgun().Properties = new Dictionary<string, string>
	{
		["v:tenant"] = "default"
	};
});
```

## Delivery Reports

This package registers a Mailgun delivery-report adaptor, so Mailgun webhook payloads can be converted into Transmitly `DeliveryReport` instances.

It also registers Mailgun delivery-report extended properties, which makes `report.Mailgun()` available when you need provider-specific webhook data.

## Related Packages

- [Transmitly](https://github.com/transmitly/transmitly)
- [Transmitly.ChannelProvider.Mailgun.Configuration](https://github.com/transmitly/transmitly-channel-provider-mailgun-configuration)
- [Transmitly.ChannelProvider.Mailgun.Api](https://github.com/transmitly/transmitly-channel-provider-mailgun-api)

---
_Copyright (c) Code Impressions, LLC. This open-source project is sponsored and maintained by Code Impressions and is licensed under the [Apache License, Version 2.0](http://apache.org/licenses/LICENSE-2.0.html)._
