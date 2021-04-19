---
title: "Enable Multiple Diagnostic Settings on Azure Resources With Arm Templates"
description: "In this blog post I will show you how to configure diagnostic settings on Azure resources using an Azure Resource Manager template."
date: 2018-02-07
draft: false
author: richard
---

In this post I will show you how to configure diagnostic settings on Azure resources using an Azure Resource Manager (ARM) template. With the diagnostic settings enabled, you can store the diagnostics in blob storage or use it other services, such as Log Analytics.

<!--more-->

## Resource diagnostic logs and diagnostic settings

Many resources in Microsoft Azure emit diagnostic logs. These logs contain operational data about the resource. Depending on the type of resource, the logs contain information about audits, counters, errors, performance metrics, etc. The diagnostic logs can be stored or used by several other services in Azure, namely Storage Accounts, Event Hubs and OMS Log Analytics. This is configured in something that's called a diagnostic 'setting'.

Last year Microsoft added the capability to configure multiple resource diagnostic settings on a single resource. This allows us to send the diagnostic data to multiple outputs simultaneously. This could be very usefull in cases where you would like to share the diagnostic data with multiple teams, who each have their own Log Analytics workspace, for example.

## Diagnostic settings in the Azure portal
The diagnostic settings of an Azure resource can be found by opening a resource in the Azure portal. In the menu on the left, under Monitoring, you'll see Diagnostic logs. Below you can see I configured two Diagnostic settings which send diagnostic logs to two different OMS Log Analytics workspaces.

{{< figure src="az-diagsettings.png" title="Diagnostic settings in Azure" >}}

## Configuring multiple diagnostic settings programmatically
The process of configuring multiple diagnostic settings on a particular Azure resource is pretty intuitive in the Azure portal. But what if you want to do this programmatically? At the moment of writing (Februari 2018), configuring multiple diagnostic settings cannot be done with the AzureRM Powershell module. The only way to accomplish this to use an [Azure Resouce Manager template](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates).

The trick is to include a child resource of the following type:

`"type": "providers/diagnosticSettings"`

The following code snippet is from a template that deploys a Logic App with two diagnostic settings. Notice the description of the Logic App and the child resources that it contains for the diagnostic settings.

```json
  "resources": [
    {
      "type": "Microsoft.Logic/workflows",
      "name": "[parameters('logicAppName')]",
      "apiVersion": "2016-10-01",
      "location": "[resourceGroup().location]",
      "properties": {
        "definition": "[parameters('logicAppDefinition')]",
        "parameters": {},
        "state": "Enabled"
      },
      "resources": [
        {
          "type": "providers/diagnosticSettings",
          "name": "[concat('Microsoft.Insights/', parameters('diagnosticSettingName01'))]",
          "apiVersion": "2017-05-01-preview",
          "properties": {
            "name": "[parameters('diagnosticSettingName01')]",
            "storageAccountId": null,
            "eventHubAuthorizationRuleId": null,
            "eventHubName": null,
            "workspaceId": "[parameters('workspaceId01')]",
            "logs": [
              {
                "category": "WorkflowRuntime",
                "enabled": true,
                "retentionPolicy": {
                  "days": 0,
                  "enabled": false
                }
              }
            ],
            "metrics": [
              {
                "timeGrain": "PT1M",
                "enabled": true,
                "retentionPolicy": {
                  "enabled": false,
                  "days": 0
                }
              }
            ]
          },
          "dependsOn": [
            "[resourceId('Microsoft.Logic/workflows', parameters('logicAppName'))]"
          ]
        },
        {
          "type": "providers/diagnosticSettings",
          "name": "[concat('Microsoft.Insights/', parameters('diagnosticSettingName02'))]",
          "apiVersion": "2017-05-01-preview",
          "properties": {
            "name": "[parameters('diagnosticSettingName02')]",
            "storageAccountId": null,
            "eventHubAuthorizationRuleId": null,
            "eventHubName": null,
            "workspaceId": "[parameters('workspaceId02')]",
            "logs": [
              {
                "category": "WorkflowRuntime",
                "enabled": true,
                "retentionPolicy": {
                  "days": 0,
                  "enabled": false
                }
              }
            ],
            "metrics": [
              {
                "timeGrain": "PT1M",
                "enabled": true,
                "retentionPolicy": {
                  "enabled": false,
                  "days": 0
                }
              }
            ]
          },
          "dependsOn": [
            "[resourceId('Microsoft.Logic/workflows', parameters('logicAppName'))]"
          ]
        }
      ]
    }
  ]
```

The diagnostic settings are configured so that they send the Logic App "WorkflowRuntime" logs and performance metrics to two separate OMS Log Analytics workspaces. For the full template, go [here](https://gist.github.com/rwaal/3d46c8b97a70750c2cbecf925b31fe92#file-azurediagsettings-json) to the gist on Github.

## How to know which logs and metrics are available?
Because each service in Azure is different, they generate different types of logs and metrics. So how do you know which logs and metrics are available for use in diagnostic settings? 

The easiest way I found is to first configure the diagnostic settings on a resource using the Azure portal. The Azure portal will display the available logs and metrics that are available. In case of a Logic App this looks like this:

{{< figure src="az-diagsettings-portal.png" title="Diagnostics settings blade in Azure portal" >}}

Then go to the Resource Group, and in the left menu, under __Settings__ click on __Automation Script__. This generates an ARM template for the resources in your Resource Group, including the diagnostic settings that you configured on your resource(s).

{{< figure src="az-diagsettings-autoscript.png" title="Automation script option in the Azure portal" >}}

## To conclude
Although Powershell support for diagnostic settings is lacking, using ARM templates is a pretty good alternative. Perhaps it's even your preferred method of deploying and configuration of your resources in Azure.
