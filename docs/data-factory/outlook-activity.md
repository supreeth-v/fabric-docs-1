---
title: Office 365 Outlook activity
description: Learn how to add an Office 365 Outlook activity to a pipeline, configure authentication and connection permissions, and send email.
ms.reviewer: xupxhou
ms.topic: how-to
ms.date: 11/23/2026
ms.custom: pipelines
ai-usage: ai-assisted
---

# Use the Office 365 Outlook activity to send an email with Outlook

The Office 365 Outlook activity in Data Factory for Microsoft Fabric lets you send emails using your Office 365 account. You can customize the message with dynamic expressions.

## Prerequisites

Make sure you have the following:

- A tenant account with an active subscription. [Sign up for free](../fundamentals/fabric-trial.md).
- A created workspace.

## Add the Office 365 Outlook activity to a pipeline

Follow these steps to use the Office 365 Outlook activity in a pipeline:

1. Create a new pipeline in your workspace.
1. Search for "Office 365 Outlook" in the pipeline **Activities** pane and select it to add it to the pipeline canvas. Depending on your browser window width, you might need to expand the activities list or look for the Outlook icon without a label.

   :::image type="content" source="media/outlook-activity/add-outlook-activity-to-pipeline.png" alt-text="Screenshot of the Fabric UI with the Activities pane and Office 365 Outlook activity highlighted.":::

1. Select the new Outlook activity on the canvas if it isn’t already selected.

   :::image type="content" source="media/outlook-activity/outlook-activity-general-settings.png" alt-text="Screenshot showing the General settings tab of the Office 365 Outlook activity.":::

Refer to the [**General** settings](activity-overview.md#general-settings) guide to configure the **General** settings tab.

> [!TIP]
> Use a separate Outlook activity for each activity you want to monitor for failures.
>
> If you attach more than one activity to the Outlook activity, all connected statuses must be met to trigger it. For example, if two copy activities are connected to an Outlook activity set to trigger "on failure," both activities must fail to activate the Outlook activity. If only one fails, the Outlook activity won’t trigger.

## Configure the Office 365 Outlook activity

To set up the Office 365 Outlook activity:

1. Add a connection to your Outlook activity at the top of the settings. You can create a new connection or use an existing one.

   > [!NOTE]
   > * If you’re using user authentication and deploying the pipeline with the Outlook or Teams activity to another workspace, the activity will be inactive in the target workspace until you create a new user authentication connection there.
   > * If you don't have access to the connections used in the Outlook activity, the initial deployment will fail. To troubleshoot, you can edit the target pipeline and change the authorization or set the activity to inactive.
   > * With user authentication (OAuth), the connection uses the authenticated user's account to send email. With Workspace Identity (WI) or Service Principal (SPN) authentication, configure the sender mailbox and grant the application identity permission to send from it. The pipeline deployer's account isn't automatically the sender.

1. Provide details for the email, including recipients, subject, and body. You can also add advanced details like a custom "from" address, CC and BCC recipients, sensitivity, and a custom reply-to address. All fields support [dynamic expressions](expression-language.md).

   :::image type="content" source="media/outlook-activity/email-details.png" alt-text="Screenshot showing the settings windows in the Outlook Activity.":::

## Share and reuse email functionality with Office 365 Outlook activity

The **Office 365 Email activity** enables pipelines to send notifications and automated messages using an authenticated connection.

In collaborative development scenarios, teams might need to reuse email logic across multiple pipelines and users. Outlook connections that use **user authentication (OAuth)** are user-specific and can't be shared directly with other authors or a deployment service principal. This restriction doesn't describe all Outlook authentication methods. For WI connection access and automated deployment, see [Connection ownership and deployment permissions](#connection-ownership-and-deployment-permissions).

To support reuse and consistency, you can adopt a pipeline-based composition pattern that centralizes email functionality. Instead of duplicating email configuration across pipelines, create a **dedicated pipeline responsible for sending emails**, and invoke it from other pipelines. In this pattern, you share the pipeline invocation connection, not the underlying user-OAuth Outlook connection.

1. Create a reusable email pipeline

   Define a pipeline that encapsulates all email-related behavior:

   - Configure the Office 365 Email activity
   - Establish the required connection
   - Validate the pipeline independently

   This pipeline acts as a reusable service for email delivery.

   :::image type="content" source="media/outlook-activity/reusable-email.png" alt-text="Screenshot showing the Settings tab and setting up a reusable Office 365 Outlook activity.":::

1. Enable shared access through a pipeline connection

   Create and configure a pipeline connection that can be used by multiple users to invoke the email pipeline via the **Manage Connections and Gateways** portal in Fabric.

   - Grant access to developers or teams who need to send emails
   - Ensure permissions align with your organization’s security policies

   :::image type="content" source="media/outlook-activity/sharing-pipeline-connection.png" alt-text="Screenshot showing how to share a Pipeline connection.":::

1. Invoke the email pipeline from other pipelines

   In downstream pipelines:
   - Add an Invoke Pipeline activity
   - Select the shared connection
   - Reference the reusable email pipeline

   This allows pipelines to trigger email sending without managing their own connections.

   :::image type="content" source="media/outlook-activity/invoke-email-activity.png" alt-text="Screenshot showing an invoke pipeline activity for the previously created email activity.":::

1. Parameterize for flexibility

   To support different use cases, define parameters in the email pipeline such as:
   - Subject
   - Message body
   - Recipients
   - Dynamic content inputs

   Passing parameters at runtime enables reuse while preserving customization.

### Authentication support

The Office 365 Outlook activity supports the following authentication methods. Authentication to Outlook is separate from permission to manage or use the Fabric connection.

| Authentication method | Connection and automation considerations |
| --- | --- |
| User authentication (OAuth) | Uses a user's credentials. The Outlook connection is user-specific and isn't directly shareable. Review the [user-authentication deployment limitations](cicd-pipelines.md#known-limitations) when moving pipelines between environments. |
| Workspace Identity (WI) | Uses a Fabric workspace identity to authenticate to Outlook. A deployment SPN can use a WI connection with the appropriate connection permissions; the SPN doesn't have to own the connection. |
| Service Principal (SPN) | Uses the application identity configured in the Outlook connection. Using SPN credentials for Outlook doesn't automatically grant that application or the deployment identity access to the Fabric connection. |

By using Workspace Identity and Service Principal support, your organization can run email notifications by using non-user identities. This approach helps reduce dependency on individual user accounts and supports production-grade automation scenarios.

### Connection ownership and deployment permissions

The following identities and permissions have different purposes:

| Identity or permission | Purpose |
| --- | --- |
| Deployment identity | Calls Fabric APIs to create or update the pipeline. It needs the applicable workspace or item permissions and access to the referenced connections. |
| Connection Owner | Manages the Fabric connection and its permissions. Ownership doesn't determine the authentication method used to access Outlook. |
| Connection User | Can use the connection without owning it. **User** is a connection permission role, not user authentication; a service principal can receive this role on a shareable connection. |
| Outlook authentication identity | Authenticates to Outlook using the credentials configured in the connection: user OAuth, WI, or SPN. |

Workspace permissions alone don't grant access to a referenced connection. For REST deployment with a WI-authenticated Outlook connection, you can use either of these arrangements:

- A user creates and owns the WI connection, then grants the deployment SPN the connection **User** role. The user remains the Owner.
- The deployment SPN creates and owns the WI connection, then deploys the pipeline that references it. No separate sharing grant to that SPN is needed.

Both arrangements use **WI authentication to Outlook**. Creating a connection as an SPN doesn't change its credentials to SPN authentication. Neither arrangement makes a user-OAuth Outlook connection shareable.

For role assignments, see [Add Connection Role Assignment](/rest/api/fabric/core/connections/add-connection-role-assignment). If the deployment SPN creates the connection, a Fabric administrator must also allow it under [Service principals can create workspaces, connections, and deployment pipelines](../admin/service-admin-portal-developer.md#service-principals-can-create-workspaces-connections-and-deployment-pipelines). This setting is separate from the setting that allows service principals to call Fabric public APIs.

### Mailbox permissions and sender

Fabric API access, connection access, and authorization to send email are separate prerequisites. When you use WI or SPN authentication, configure the appropriate Microsoft Entra ID and Microsoft Graph application permissions, obtain administrator consent, and specify a mailbox that the identity is authorized to send from. The sender mailbox must be available in Exchange Online and support the required mail API operations. Depending on your organizational requirements, you might also need to set up additional Exchange Online mailbox restrictions and governance policies.

   > [!NOTE]
   > Service Principal (SPN) and Workspace Identity (WI) use application-level Microsoft Graph permissions to access Outlook. A tenant administrator must explicitly grant the required permissions and provide admin consent before the identity can be used. These permissions can allow the identity to send email on behalf of users in the tenant. Before granting access, tenant administrators should review the permission scope and ensure it aligns with their organization's security policies.

For more information about setting up WI or SPN, see [Workspace identity support in Data Factory](workspace-identity.md) or [Service principal support in Data Factory](service-principals.md).

Successful deployment or connection access doesn't by itself verify email delivery. Run the activity and verify delivery to the intended recipients after configuring the mailbox and permissions.

## Known limitations

- The Outlook activity doesn't support sensitivity labels. You can't apply Microsoft Purview sensitivity labels to emails sent through the activity.

## Related content

- [Run, schedule, and trigger pipelines](pipeline-runs.md)
- [Monitor pipeline runs](monitor-pipeline-runs.md)
