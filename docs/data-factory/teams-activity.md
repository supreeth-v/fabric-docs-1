---
title: Teams activity
description: Learn how to add a Teams activity to a pipeline and use it to send a Teams message.
ms.reviewer: xupxhou
ms.topic: how-to
ms.custom: pipelines
ms.date: 11/23/2026
ai-usage: ai-assisted
---

# Send Teams messages with the Teams activity

The Teams activity in Data Factory for Microsoft Fabric lets you send messages to Teams channels or group chats. You can customize messages with dynamic expressions to include information from your pipeline.

## Prerequisites

* A tenant account with an active subscription. [Create an account for free](../fundamentals/fabric-trial.md).
* A workspace.

## Add Teams activity to a pipeline

To integrate Teams messaging into your pipeline, you need to add the Teams activity. This section guides you through the steps to include and configure the Teams activity in your pipeline.

1. Create a new pipeline in your workspace.
1. In the **Activities** pane, search for Teams and select it to add it to the pipeline canvas.

   :::image type="content" source="media/teams-activity/add-teams-activity-to-pipeline.png" alt-text="Screenshot of the Fabric UI with the Activities pane and Teams activity highlighted.":::

1. Select the new Teams activity on the canvas.

   :::image type="content" source="media/teams-activity/teams-activity-general-settings.png" alt-text="Screenshot showing the General settings tab of the Teams activity.":::

For details about the **General** settings tab, see [General settings](activity-overview.md#general-settings).

## Configure Teams activity settings

Set up the Teams activity by following these steps:

1. Add a connection to your Teams activity. You can create a new connection or use an existing one.

    > [!NOTE]
    > * If you're using user authentication and deploying the pipeline with the Teams activity to another workspace, the activity will be inactive until you create a new user authentication connection in the target workspace.
    > * If you don't have access to the connections used in the Teams activity, the initial deployment will fail. To troubleshoot, you can edit the target pipeline and change the authorization or set the activity to inactive.

   :::image type="content" source="media/teams-activity/connect-to-teams.png" alt-text="Screenshot showing the Settings tab of the Teams activity highlighting where to create a new connection or use an existing connection.":::

1. Choose where to post the message using the **Post in** setting. You can post to a channel or a group chat.

   :::image type="content" source="media/teams-activity/post-in-channel-or-group-chat.png" alt-text="Screenshot showing the Settings tab of the Teams activity highlighting where to select posting to a channel or group chat.":::

1. If you select group chat, use the dropdown to pick a group chat you're part of. If you select channel, choose a team and then a channel from the dropdowns.

1. Write your message in the **Message** area. Use [dynamic expressions](expression-language.md) to include system or user variables, expressions, or functions. Select **View in expression builder** to add dynamic expressions.

   :::image type="content" source="media/teams-activity/create-message.png" alt-text="Screenshot showing the Settings tab of the Teams activity highlighting the link to select to use dynamic expressions.":::

1. If you're posting to a channel, you can add a subject for the message in the **Subject** text box.

   :::image type="content" source="media/teams-activity/select-a-subject.png" alt-text="Screenshot showing the Settings tab of the Teams activity highlighting boxes to add the subject.":::

## Legacy Teams activity

Starting October 30, 2026, you can't add new legacy Teams activities to pipelines. Existing legacy Teams activities can continue to run and be edited. This creation cutoff doesn't require you to migrate existing activities immediately.

If a retirement date for existing legacy activities is finalized, the [lifecycle announcement](https://community.fabric.microsoft.com/blog/fbc_fabricupdatesblogs/upcoming-changes-to-legacy-teams-and-outlook-activities-in-fabric-data-factory-p/5314499) will be updated. Refer to that announcement for changes to the timeline.

For new implementations, use the Teams activity described in this article. Follow the connection and deployment requirements in the following sections when planning a move from the legacy activity.

## Known limitations

- The Teams activity will be inactive when using CI/CD. See [CI/CD deployment considerations](#cicd-deployment-considerations) for target-workspace configuration.

### Connection authentication and ownership

The Microsoft Teams activity uses user-based authentication. Each Teams connection is associated with the identity and authentication token of the user who created it.

The following considerations apply when configuring Teams connections:

- **User-specific connections:** Each user who wants to run a pipeline that sends Teams messages must create and authenticate their own Teams connection.
- **Connection sharing:** Teams connections can't be shared across users or Microsoft Entra security groups. Workspace role assignments and group membership don't grant access to another user's Teams connection. Creating a connection by using an administrator account doesn't allow other users to reuse it.
- **Workspace configuration:** When deploying pipelines across workspaces, ensure that the appropriate Teams connection is configured and authenticated for the user running the pipeline.
- **Production environments:** Teams activity requires user authentication and doesn't currently support service principal (SPN) or workspace identity (WI) authentication. Production pipelines that use Teams activity must therefore rely on a user-authenticated connection.
- **Service accounts:** A service account that signs in as a user still uses user authentication. It doesn't make the connection shareable or enable SPN or WI authentication, and it remains subject to account policies and reauthentication requirements.

> [!NOTE]
> Teams connections are tied to the authenticated user's identity, regardless of whether the account has administrator privileges. For production scenarios, consider your organization's identity management and governance requirements when determining which account will create and maintain the connection.

### CI/CD deployment considerations

The identity that deploys a pipeline and the identity that authenticates its Teams connection serve different purposes. Support for service principal calls to Fabric APIs doesn't enable service principal authentication to Teams or grant the service principal access to another user's Teams connection. Don't treat pipeline deployment permissions as Teams connection permissions.

For the user-authenticated Teams activity, deployment doesn't remove the target-side authentication requirement. After deploying to another workspace:

1. Open the target pipeline and select the Teams activity.
1. Create and authenticate the appropriate user connection in the target workspace, and select it for the activity.
1. Verify the destination team and channel or the group chat, and review the message settings. Ensure the activity is active, save the pipeline, and run it to verify message delivery before enabling production schedules.

This process includes manual user authentication; it isn't a fully unattended, app-only deployment and execution pattern. For the documented OAuth deployment limitations, see [CI/CD for pipelines in Data Factory](cicd-pipelines.md#known-limitations).

## Related content

- [CI/CD for pipelines in Data Factory](cicd-pipelines.md)
- [Run, schedule, and trigger pipelines](pipeline-runs.md)
- [Monitor pipeline runs](monitor-pipeline-runs.md)
