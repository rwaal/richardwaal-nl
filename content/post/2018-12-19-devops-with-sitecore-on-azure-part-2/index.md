---
title: "DevOps with Sitecore on Azure Part 2 - Using SQL Elastic Pools with Sitecore"
description: "This is the second of two posts with tips and tricks about running Sitecore on PaaS services in Azure. Today I will show how to use SQL Elastic Pools with Sitecore."
date: 2018-12-29
draft: false
author: richard

tags: [
    "app services",
    "arm templates",
    "azure devops",
    "azure sql",
    "sitecore"
]
---

This blog post is the fist of two posts with tips and tricks about running Sitecore on Azure PaaS services. Today I will show you how you can deploy Sitecore 9 to an [App Service Environment](https://docs.microsoft.com/en-us/azure/app-service/environment/intro) (ASE).

<!--more-->

## App Service Environment

The ARM templates that are provided by Sitecore, through their [Github repo](https://github.com/Sitecore/Sitecore-Azure-Quickstart-Templates/), deploy Sitecore on so-called multi-tenant App Services. With a few modifications to these templates you can deploy the Sitecore App Service Plans to an App Service Environment. This provides network isolation, connections to on-premises systems through Express Route and more.

The modifications that I describe in this post should work for all Sitecore 9.x configurations. Please note, the ASE must already exist for this to work.

### parameters.json file

I begin by defining two parameters in the azuredeploy.parameters.json file, called "aseName" and "aseResourceGroupName". The first specifies the name of the ASE and the second specifies the Resource Group name that the ASE resides in.

```json
"aseName": {
    "value": "myDummyAse"
},
"aseResourceGroupName": {
    "value": "myAseRg"
},
```

### azuredeploy.json file

The azuredeploy.json file doesn't create any Azure resources itself. It functions as the main template from which linked or nested templates are deployed. We need to make a few modifications to this template.

__Step 1__: Define the two parameter we just create in the parameters section of the template.

```json
"aseName": {
    "type": "string",
    "metadata": {
    "description": "Name of the App Service Environment"
    }
},
"aseResourceGroupName": {
    "type": "string",
    "metadata": {
    "description": "Name of the resource group that the ASE resides in."
    }
},
```

__Step 2__: Add the parameter to the resource where the nested infrastructure.json template is referenced: "[concat(parameters('deploymentId'), '-infrastructure')]"

```json
"resources": [
{
    "apiVersion": "[variables('resourcesApiVersion')]",
    "name": "[concat(parameters('deploymentId'), '-infrastructure')]",
    "type": "Microsoft.Resources/deployments",
    "properties": {
    "mode": "Incremental",
    "templateLink": {
        "uri": "[concat(uri(parameters('templateLinkBase'), 'nested/infrastructure.json'), parameters('templateLinkAccessToken'))]"
    },
    "parameters": {
        "deploymentId": {
        "value": "[parameters('deploymentId')]"
        },
        "location": {
        "value": "[parameters('location')]"
        },

        "aseName": {
        "value": "[parameters('aseName')]"
        },
        "aseResourceGroupName": {
        "value": "[parameters('aseResourceGroupName')]"
        },
```

__Step 3__: Add the parameter to the other infrastructure template references, like  -infrastructure-exm.json or -infrastructure-ma.json. Which infrastructure template resources are in the file depends on the Sitecore topology that you are deploying.

For each of the infrastructure template resources, add the parameter exactly the same as in Step 2.

### infrastructure.json file

__Step 1__: Add the parameters "aseName" and "aseResourceGroupName"to the parameter section, just like you did in Step 1 of the azuredeploy.json.

__Step 2__: Change the properties of the "Microsoft.Web/serverfarms" resouces. Simply do a search for all resources of type "Microsoft.Web/serverfarms" and make sure the hostingEnvironment and hostingEnvironmentId properties are configured as below.

```json
{
    "type": "Microsoft.Web/serverfarms",
    "name": "[variables('singleHostingPlanNameTidy')]",
    "apiVersion": "[variables('serverFarmApiVersion')]",
    "sku": {
    "name": "[parameters('singleHostingPlanSkuName')]",
    "capacity": "[parameters('singleHostingPlanSkuCapacity')]"
    },
    "properties": {
    "name": "[variables('singleHostingPlanNameTidy')]",
    "hostingEnvironment": "[parameters('aseName')]",
    "hostingEnvironmentId": "[resourceId(parameters('aseResourceGroupName'),'Microsoft.Web/hostingEnvironments', parameters('aseName'))]"        
    },
    "location": "[parameters('location')]",
    "tags": {
    "provider": "[variables('sitecoreTags').provider]",
    "logicalName": "[variables('sitecoreTags').single]"
    }
},
```

### Other infrastructure files

Depending on the Sitecore configuration you are deploying, you need to repeat the steps listed in infrastructure.json for othter templates. For example, when you are deploying the Sitecore XP Scaled configuration, you also need to modify the files infrastructure-exm.json, infrastructure-ma.json and infrastructure-xc.json.

Simply look inside the /nested folder and change all the templates that contain the term "infrastructure" and where resources are created of type Microsoft.Web/serverfarms.

## More information

Check out [my Github repository](https://github.com/rwaal/SitecoreDevOpsOnAzure) to find an example of an XP Single setup in an App Service Environment.

I would also highly recommend the "[Sitecore 9.0.1 on Azure: PaaS Deployment Guide](https://sitecorehacker.com/2018/01/18/sitecore-9-0-1-on-azure-paas-deployment-guide/)" article by Pete Navarra, where he explains how to get started with deploying Sitecore on Azure.
