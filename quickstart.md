# Quickstart Guide

The following guide will walk you through sending out your first campaign. It assumes that you have:

- successfully installed and configured SendPortal; and
- created your first user; and
- that you are logged in.

For more detailed instructions for each feature, refer to the relevant article in the Features section.

## Create An Email Service

SendPortal integrates with Mailgun, Amazon SES, Sendgrid and Postmark.

> As per their [Terms of Service](https://postmarkapp.com/terms-of-service#email-types-that-we-dont-allow-on-postmark), Postmark should only be used for transactional mail, and not for marketing messages.

Click on the _Email Services_ menu and then click the _Add Email Service_ button.

The required information will vary between each service. Full instructions on how to set up each service can be found in the Email Service documentation.

## Add Or Import Subscribers

In order for SendPortal to send emails, you need to set up lists of [Subscribers](/docs/subscribers). These subscribers can be added in 3 ways:

- Manually
- Through the API
- CSV Import

Refer to the Subscriber documentation for more information.

## Create A Template

A [Template](/docs/templates) allows you to create consistent branding for all your campaigns, so that you only need to create new content for each campaign.

> Campaigns can be created without a template as well, if desired.

To create a template, click on the _Templates_ menu and then click the _Add Template_ button. Templates should contain valid HTML and must include at least a `{{content}}` tag.

Refer to the [Templates documentation](/docs/templates) for more information.

## Create A Campaign

To create a campaign, click on the _Campaigns_ menu and then click the _New Campaign_ button.

Make sure to select the template you created in the previous step.

The content will be placed into the `{{content}}` tag in the template.

Refer to the [Campaign documentation](/docs/campaigns) for more information.

## Send The Campaign

After your campaign has been created, you will have a chance to review it before sending.

#### Test Email

You can test the email by entering an address in the Test Email section and clicking _Send Test Email_.

#### Recipients

Specify who the campaign should be sent to; either **All Subscribers,** or one or more **Segments**.

#### Schedule

Choose to dispatch the campaign immediately or at a specific and date and time.

#### Sending Behaviour

Finally, you can choose to save each individual email as a draft message, which means you will will need to review and send each individual Message before it is sent out. Alternatively, you can choose to send all the messages immediately.

## Next Steps

This covers the basics of how to set up your Providers, Templates, Subscribers and Campaigns.

For more detailed information on each of these items, please refer to the Features section.