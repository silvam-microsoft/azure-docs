---
title: Create automation tasks to manage and monitor Azure resources
description: Set up automated tasks that help you manage Azure resources and monitor costs by creating workflows that run on Azure Logic Apps.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, azla
ms.topic: how-to
ms.date: 11/02/2021
ms.custom: ignite-fall-2021
---

# Manage Azure resources and monitor costs by creating automation tasks (preview)

> [!IMPORTANT]
> This capability is in preview and is subject to the 
> [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

To help you manage [Azure resources](../azure-resource-manager/management/overview.md#terminology) more easily, you can create automated management tasks for a specific resource or resource group by using automation task templates, which vary in availability based on the resource type. For example, for an [Azure storage account](../storage/common/storage-account-overview.md), you can set up an automation task that sends you the monthly cost for that storage account. For an [Azure virtual machine](https://azure.microsoft.com/services/virtual-machines/), you can create an automation task that turns on or turns off that virtual machine on a predefined schedule.

Here are the currently available task templates in this preview:

| Resource type | Automation task templates |
|---------------|---------------------------|
| Azure resource groups | **When resource is deleted** |
| All Azure resources | **Send monthly cost for resource** |
| Azure virtual machines | Additionally: <p>- **Power off Virtual Machine** <br>- **Start Virtual Machine** |
| Azure Storage accounts | Additionally: <p>- **Delete old blobs** |
| Azure Cosmos DB | Additionally, <p>- **Send query result via email** |
|||

This article shows you how to complete the following tasks:

* [Create an automation task](#create-automation-task) for a specific Azure resource.

* [Review a task's history](#review-task-history), which includes the run status, inputs, outputs, and other historical information.

* [Edit the task](#edit-task) so that you can update the task, or customize the task's underlying workflow in the workflow designer.

<a name="differences"></a>

## How do automation tasks differ from Azure Automation?

Automation tasks are more basic and lightweight than [Azure Automation](../automation/automation-intro.md). Currently, you can create an automation task only at the Azure resource level. Behind the scenes, an automation task is actually a logic app resource that runs a workflow and is powered by the [*multi-tenant* Azure Logic Apps service](logic-apps-overview.md). After you create the automation task, you can view and edit the underlying workflow by opening the task in the workflow designer. After a task finishes at least one run, you can review the task's status, workflow run history, inputs, and outputs for each run.

By comparison, Azure Automation is a cloud-based automation and configuration service that supports consistent management across your Azure and non-Azure environments. The service comprises [process automation for orchestrating processes](../automation/automation-intro.md#process-automation) by using [runbooks](../automation/automation-runbook-execution.md), configuration management with [change tracking and inventory](../automation/change-tracking/overview.md), update management, shared capabilities, and heterogeneous features. Automation gives you complete control during deployment, operations, and decommissioning of workloads and resources.

<a name="pricing"></a>

## Pricing

When you create an automation task, charges don't start automatically incurring. Underneath, an automation task is powered by a workflow in a logic app resource that's hosted in multi-tenant Azure Logic Apps. So, the [Consumption pricing model](logic-apps-pricing.md) applies to automation tasks. Metering and billing are based on the trigger and action executions in the underlying logic app workflow.

Executions are metered and billed, regardless whether the workflow runs successfully or whether the workflow is even instantiated. For example, suppose your automation task uses a polling trigger that regularly makes an outgoing call to an endpoint. This outbound request is metered and billed as an execution, regardless whether the trigger fires or is skipped, which affects whether a workflow instance is created.

Triggers and actions follow [Consumption plan rates](https://azure.microsoft.com/pricing/details/logic-apps/), which differ based on whether these operations are ["built-in"](../connectors/built-in.md) or ["managed" (Standard or Enterprise)](../connectors/managed.md). Triggers and actions also make storage transactions, which use the [Consumption plan data rate](https://azure.microsoft.com/pricing/details/logic-apps/).

> [!TIP]
> As a monthly bonus, the Consumption plan includes *several thousand* built-in executions free of charge. 
> For specific information, review the [Consumption plan rates](https://azure.microsoft.com/pricing/details/logic-apps/).

## Prerequisites

* An Azure account and subscription. If you don't have a subscription, [sign up for a free Azure account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* The Azure resource that you want to manage. This article uses an Azure storage account as the example.

* An Office 365 account if you want to follow along with the example, which sends you email by using Office 365 Outlook.

<a name="create-automation-task"></a>

## Create an automation task

1. In the [Azure portal](https://portal.azure.com), find the resource that you want to manage.

1. On the resource navigation menu, in the **Automation** section, and select **Tasks (preview)**.

   ![Screenshot that shows the Azure portal and storage account resource menu with "Tasks (preview)" selected.](./media/create-automation-tasks-azure-resources/storage-account-menu-automation-section.png)

1. On the **Tasks** pane, select **Add a task** so that you can select a task template.

   ![Screenshot that shows the "Tasks (preview)" pane with "Add a task" selected.](./media/create-automation-tasks-azure-resources/add-automation-task.png)

1. On the **Add a task** pane, under **Select a template**, in the template for the replication task that you want to create, select **Select**. If the next page doesn't appear, select **Next: Authenticate**.

   This example continues by selecting the **Send monthly cost for resource** task template.

   ![Screenshot that shows the "Add a task" pane with "Send monthly cost for resource" template selected.](./media/create-automation-tasks-azure-resources/select-task-template.png)

1. Under **Authenticate**, in the **Connections** section, select **Create** for every connection that appears in the task so that you can provide authentication credentials for all the connections. The types of connections in each task vary based on the task.

   This example shows only one of the connections that's required by this task.

   ![Screenshot that shows the selected "Create" option for the Azure Resource Manager connection.](./media/create-automation-tasks-azure-resources/create-authenticate-connections.png)

1. When you're prompted, sign in with your Azure account credentials.

   ![Screenshot that shows the selection, "Sign in".](./media/create-automation-tasks-azure-resources/create-connection-sign-in.png)

   Each successfully authenticated connection looks similar to this example:

   ![Screenshot that shows successfully created connection.](./media/create-automation-tasks-azure-resources/create-connection-success.png)

1. After you authenticate all the connections, select **Next: Configure** if the next page doesn't appear.

1. Under **Configure**, provide a name for the task and any other information required for the task. When you're done, select **Review + create**.

   > [!NOTE]
   > You can't change the task name after creation, so consider a name that still applies if you 
   > [edit the underlying workflow](#edit-task-workflow). Changes that you make to the underlying 
   > workflow apply only to the task that you created, not the task template.
   >
   > For example, if you name your task `SendMonthlyCost`, but you later edit the underlying workflow 
   > to run weekly, you can't change your task's name to `SendWeeklyCost`.

   Tasks that send email notifications require an email address.

   ![Screenshot that shows the required information for the selected task.](./media/create-automation-tasks-azure-resources/provide-task-information.png)

   The task that you created, which is automatically live and running, now appears on the **Tasks** list.

   ![Screenshot that shows the automation tasks list](./media/create-automation-tasks-azure-resources/automation-tasks-list.png)

   > [!TIP]
   > If the task doesn't appear immediately, try refreshing the tasks list or wait a little before you refresh. On the toolbar, select **Refresh**.

   After the selected task runs, you get an email that looks like this example:

   ![Screenshot that shows email notification sent by task.](./media/create-automation-tasks-azure-resources/email-notification-received.png)

<a name="review-task-history"></a>

## Review task history

To view a task's history of runs along with their statuses, inputs, outputs, and other information, follow these steps:

1. In the [Azure portal](https://portal.azure.com), find the resource that has the task history that you want to review.

1. On the resource's menu, under **Settings**, in the **Automation** section, select **Tasks (preview)**.

1. In the tasks list, find the task that you want to review. In that task's **Runs** column, select **View**.

   ![Screenshot that shows a task and the selected "View" option.](./media/create-automation-tasks-azure-resources/view-runs-for-task.png)

   The **Runs history** pane shows all the runs for the task along with their statuses, start times, identifiers, and run durations.

   ![Screenshot that shows a task's runs, their statuses, and other information.](./media/create-automation-tasks-azure-resources/view-runs-history.png)

   Here the possible statuses for a run:

   | Status | Description |
   |--------|-------------|
   | **Cancelled** | The task was canceled while running. |
   | **Failed** | The task has at least one failed action, but no subsequent actions existed to handle the failure. |
   | **Running** | The task is currently running. |
   | **Succeeded** | All actions succeeded. A task can still finish successfully if an action failed, but a subsequent action existed to handle the failure. |
   | **Waiting** | The run hasn't started yet and is paused because an earlier instance of the task is still running. |
   |||

   For more information, see [Review runs history in monitoring view](monitor-logic-apps.md#review-runs-history).

1. To view the statuses and other information for each step in a run, select that run.

   The **Logic app run** pane opens and shows the underlying workflow that ran.

   * A workflow always starts with a [*trigger*](../connectors/apis-list.md#triggers). For this task, the workflow starts with the [**Recurrence** trigger](../connectors/connectors-native-recurrence.md).

   * Each step shows its status and run duration. Steps that have 0-second durations took less than 1 second to run.

   ![Screenshot that shows each step in the run, status, and run duration.](./media/create-automation-tasks-azure-resources/runs-history-details.png)

1. To review the inputs and outputs for each step, select the step, which expands.

   This example shows the inputs for the Recurrence trigger, which has no outputs because the trigger only specifies when the workflow runs and provides no outputs for the subsequent actions to process.

   ![Screenshot that shows the expanded trigger and inputs.](./media/create-automation-tasks-azure-resources/view-trigger-inputs.png)

   In contrast, the **Send an email** action has inputs from earlier actions in the workflow and outputs.

   ![Screenshot that shows an expanded action, inputs, and outputs.](./media/create-automation-tasks-azure-resources/view-action-inputs-outputs.png)

To learn how you can build your own automated workflows so that you can integrate apps, data, services, and systems apart from the context of automation tasks for Azure resources, see [Quickstart: Create your first integration workflow by using Azure Logic Apps - Azure portal](quickstart-create-first-logic-app-workflow.md).

<a name="edit-task"></a>

## Edit the task

To change a task, you have these options:

* [Edit the task "inline"](#edit-task-inline) so that you can change the task's properties, such as connection information or configuration information, for example, your email address.

* [Edit the task's underlying workflow](#edit-task-workflow) in the workflow designer.

<a name="edit-task-inline"></a>

### Edit the task inline

1. In the [Azure portal](https://portal.azure.com), find the resource that has the task that you want to update.

1. On the resource navigation menu, in the **Automation** section, select **Tasks (preview)**.

1. In the tasks list, find the task that you want to update. Open the task's ellipses (**...**) menu, and select **Edit in-line**.

   ![Screenshot that shows the opened ellipses menu and the selected option, "Edit in-line".](./media/create-automation-tasks-azure-resources/edit-task-in-line.png)

   By default, the **Authenticate** tab appears and shows the existing connections.

1. To add new authentication credentials or select different existing authentication credentials for a connection, open the connection's ellipses (**...**) menu, and select either **Add new connection** or if available, different authentication credentials.

   ![Screenshot that shows the Authentication tab, existing connections, and the selected ellipses menu.](./media/create-automation-tasks-azure-resources/edit-connections.png)

1. To update other task properties, select **Next: Configure**.

   For the task in this example, the only property available for edit is the email address.

   ![Screenshot that shows the "Configure" tab.](./media/create-automation-tasks-azure-resources/edit-task-configuration.png)

1. When you're done, select **Save**.

<a name="edit-task-workflow"></a>

### Edit the task's underlying workflow

When you change the underlying workflow for an automation task, your changes affect only the task instance that you created, and not the template that creates the task. After you make and save your changes, the name that you provided for your original task might not accurately describe the task anymore, so you might have to recreate the task with a different name.

> [!TIP]
> As a best practice, clone the underlying workflow so that you can edit the copied version instead. 
> That way, you can make and test your changes on the copy while the original automation task continues 
> to work and run without risking disruption or breaking existing functionality. After you finish your 
> changes and are satisfied that the new version runs successfully, you can disable or delete the 
> original automation task and use the cloned version for your automation task. The following steps 
> include information about how to clone your workflow.

1. In the [Azure portal](https://portal.azure.com), find the resource that has the task that you want to update.

1. On the resource navigation menu, in the **Automation** section, select **Tasks**.

1. In the tasks list, find the task that you want to update. Open the task's ellipses (**...**) menu, and select **Open in Logic Apps**.

   ![Screenshot that shows the opened ellipses menu and the selected option, "Open in Logic Apps".](./media/create-automation-tasks-azure-resources/edit-task-logic-app-designer.png)

   The task's underlying workflow opens in the Azure Logic Apps service and shows the **Overview** pane where you can view the same runs history that's available for the task.

   ![Screenshot that shows the task in Azure Logic Apps view with Overview pane selected.](./media/create-automation-tasks-azure-resources/task-logic-apps-view.png)

1. To open the underlying workflow in the designer, on the logic app navigation menu, select **Logic app designer**.

   ![Screenshot that shows the "Logic app designer" menu option selected and designer surface with the underlying workflow.](./media/create-automation-tasks-azure-resources/view-task-workflow-logic-app-designer.png)

   You can now edit the properties for the workflow's trigger and actions as well as edit the trigger and actions that define the workflow itself. However, as best practice, follow the steps to clone your workflow so that you can make your changes on a copy while the original workflow continues to work and run.

1. To clone your workflow and edit the copied version instead, follow these steps:

   1. On the logic app workflow menu, select **Overview**.

   1. On the overview pane's toolbar, select **Clone**.

   1. On the logic app creation pane, under **Name**, enter a new name for your copied logic app workflow.

      Except for **Logic App Status**, the other properties aren't available for editing.

   1. Under **Logic App Status**, select **Disabled** so that the cloned workflow doesn't run while you make your changes. You can enable the workflow when you're ready to test your changes.

   1. After Azure finishes provisioning your cloned workflow, find and open that workflow in the designer.

1. To view the properties for the trigger or an action, expand that trigger or action.

   For example, you can change the Recurrence trigger to run weekly, rather than monthly.

   ![Screenshot that shows the expanded Recurrence trigger with the Frequency list open to show available frequency options.](./media/create-automation-tasks-azure-resources/edit-recurrence-trigger.png)

   For more information about the Recurrence trigger, see [Create, schedule, and run recurring tasks and workflows with the Recurrence trigger.](../connectors/connectors-native-recurrence.md). For more information about other triggers and actions that you can use, see [Connectors for Azure Logic Apps](../connectors/apis-list.md).

1. To save your changes, on the designer toolbar, select **Save**.

   ![Screenshot that shows the designer toolbar and the selected "Save" command.](./media/create-automation-tasks-azure-resources/save-updated-workflow.png)

1. To test and run the updated workflow, on the designer toolbar, select **Run**.

   After the run finishes, the designer shows the workflow's run details.

   ![Screenshot that shows the workflow's run details on the designer.](./media/create-automation-tasks-azure-resources/view-run-details-designer.png)

1. To disable the workflow so that the task doesn't continue running, see [Manage logic apps in the Azure portal](../logic-apps/manage-logic-apps-with-azure-portal.md).

## Provide feedback

We'd like to hear from you! To report bugs, provide feedback, or ask questions about this preview capability, [contact the Azure Logic Apps team](mailto:logicappspm@microsoft.com).

## Next steps

* [Manage logic apps in the Azure portal](manage-logic-apps-with-azure-portal.md)
