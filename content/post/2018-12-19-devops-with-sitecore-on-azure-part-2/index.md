---
title: "DevOps with Sitecore on Azure Part 2 - Using SQL Elastic Pools with Sitecore"
description: "This is the second of two posts with tips and tricks about running Sitecore on PaaS services in Azure. Today I will show how to use SQL Elastic Pools with Sitecore."
date: 2018-12-29
draft: false
author: richard
thumbnail: /posts/elastic-pool.png
featureImage: /posts/elastic-pool.png

tags: [
    "app services",
    "arm templates",
    "azure devops",
    "azure sql",
    "sitecore"
]
---

This is the second of two posts with tips and tricks about running Sitecore on PaaS services in Azure. Today I will show how to use SQL Elastic Pools with Sitecore.

<!--more-->

## Introduction
Azure SQL Elastic Database Pools allow you to group your databases and use a shared pool of resources. Instead of assigning a fixed set of performance capabilities to a single database (measured in [DTU](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-service-tiers-dtu)), you assign the capabilities to the shared pool.

Elastic Pools are very useful with databases that have varying, unpredictable or spiky usage patterns. A database can burst in performance and take the resources it needs from the pool. This makes it very useful in a Sitecore implementation on Azure. In this blog post I will describe how you can modify your ARM templates to deploy Sitecore 9 in Azure using an Sql Elastic Pool.

## When to use Elastic Pools?
Elastic Pools are very useful when you have a collection of databases with a relatively low average utilization, but that have occasional spikes in the capacity that they require.

The image below, taken from the Microsoft docs website, shows an example where you have an Elastic Pool that has a pool size of 100 DTU. The image shows that usage patterns differs per database, and that they each can take the capacity of the shared resources to burst.

{{< figure src="elastic-pool-databases-usage.png" title="Usage patterns in Elastic Pool" >}}

In my experience many Sitecore setups show these kind of usage patterns. When you follow the [Sitecore guidelines](https://kb.sitecore.net/articles/043375) for sizing your Azure resources, I found that you often run in the situation where the databases spike in utilization and become a bottleneck.

You could simply scale up your individual databases to a higher SKU without using an Elastic Pool. But then you might be paying for capacity that you only need occasionally. On the other hand, Elastic Pools are generally a bit more expensive. But I found that the additional costs are worth it. Especially in acceptance or production environments I often choose to implement an Elastic Pool.

When determining the pool size in the Elastic Pool, I usually just add up the DTU’s per database as listed in the Sitecore sizing recommendations to determine how many DTU’s I need for all the databases. Then I look at the Elastic Pool size that best matches the DTU’s that I need in total. By the way, I usually round the number down because some of the Sitecore databases require very little capacity and infrequently spike.

## The ARM templates

ARM templates are used to deploy Sitecore on Azure. We will be using the ARM templates that are provided by Sitecore in [their Github repository](https://github.com/Sitecore/Sitecore-Azure-Quickstart-Templates). We need to make changes to several ARM templates in order to create the Elastic Pool and to link the Sql databases to the it.

In this article I am deploying Sitecore 9 to Azure. And in order to make use of an Sql Elastic Pool you need to change the following templates, when deploying either XM Scaled, XM Single, XP Scaled or SX Single.

* azuredeploy.parameters.json
* azuredeploy.json
* infrastructure.json

### azuredeploy.parameters.json

I am defining two parameters in the azuredeploy.parameters.json file related to the Elastic Pool. Parameter “sqlElasticPoolName” provides the name of the Elastic Pool resource and “sqlElasticPoolLimit” determines the amount of DTU’s that you are assigning to the pool.

```
"sqlElasticPoolName":{
  "value": ""
},
"sqlElasticPoolLimit": {
  "value": ""
},
```

### azuredeploy.json file

__Step 1__: Defining the Elastic Pool related parameters in the azuredeploy.son file. In my case, I have defined five parameters in the azuredeploy.json.

```
"sqlElasticPoolName": {
    "type": "string",
    "minLength": 1,
    "defaultValue": "[concat(parameters('deploymentId'), '-sqlpool')]"
},
"sqlElasticPoolskuName": {
    "type": "string",
    "minLength": 1,
    "defaultValue": "StandardPool"
},
"sqlElasticPoolTier": {
    "type": "string",
    "minLength": 1,
    "defaultValue": "Standard"
},
"sqlElasticPoolLimit": {
    "type": "string",
    "defaultValue": "100"
},
"sqlElasticPoolStorageSize": {
    "type": "int",
    "minValue": 1000000000,
    "defaultValue": 50000000000
},
```

You might notice that I didn’t reference all of the parameters in the parameter file. This is because I have assigned default values that I will probably always remain unchanged.

__Step 2__: Add the parameter to the resource where the nested infrastructure.json template is referenced: “[concat(parameters(‘deploymentId’), ‘-infrastructure’)]”

```
"sqlElasticPoolName": {
    "value": "[parameters('sqlElasticPoolName')]"
},
"sqlElasticPoolskuName": {
    "value": "[parameters('sqlElasticPoolskuName')]"
},
"sqlElasticPoolTier": {
    "value": "[parameters('sqlElasticPoolTier')]"
},
"sqlElasticPoolLimit": {
    "value": "[parameters('sqlElasticPoolLimit')]"
},
"sqlElasticPoolStorageSize": {
    "value": "[parameters('sqlElasticPoolStorageSize')]"
},
```
### infrastructure.json file

__Step 1__: Add the five parameters that we defined in the azuredeploy.json file in the infrastructure.json file. You can simply select and copy those parameters from the parameters section in azurdeploy.json to infrastructure.json.

__Step 2__: We need to add some json to actually create the Sql Elastic Pool. In the snippet below you can see that I created a (child) resouce.

```
    {
      "type": "Microsoft.Sql/servers",
      "apiVersion": "[variables('dbApiVersion')]",
      "properties": {
        "administratorLogin": "[parameters('sqlServerLogin')]",
        "administratorLoginPassword": "[parameters('sqlServerPassword')]",
        "version": "[parameters('sqlServerVersion')]"
      },
      "name": "[variables('sqlServerNameTidy')]",
      "location": "[parameters('location')]",
      "tags": {
        "provider": "[variables('sitecoreTags').provider]"
      },
      "resources": [
        {
          "apiVersion": "2017-10-01-preview",
          "dependsOn": [
            "[resourceId('Microsoft.Sql/servers', variables('sqlServerNameTidy'))]"
          ],
          "location": "[parameters('location')]",
          "name": "[concat(variables('sqlServerNameTidy'), '/', parameters('sqlElasticPoolName'))]",
          "sku": {
            "name": "[parameters('sqlElasticPoolskuName')]",
            "tier": "[parameters('sqlElasticPoolTier')]",
            "capacity": "[parameters('sqlElasticPoolLimit')]"
          },
          "properties": {
            "licenseType": "",
            "storageMB": "[parameters('sqlElasticPoolStorageSize')]"
          },
          "type": "Microsoft.Sql/servers/elasticpools"
        },
```

Please note that you have the option to configure a minimum capacity all databases are guaranteed, and the maximum capacity any one database can consume. This can be relevant if you have a database that is using all the resources without leaving anything for other databases. The snippet above doesn’t use these options. For more information about limits, see [this article](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-dtu-resource-limits-elastic-pools), and [this page](https://docs.microsoft.com/en-us/azure/templates/microsoft.sql/2017-10-01-preview/servers/elasticpools) for the ARM template reference.

## More information

You can find the complete set of ARM templates that I created for this article in [my Github repository](https://github.com/rwaal/SitecoreDevOpsOnAzure). Also check out [this](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-elastic-pool) Microsoft docs page for a detailed description about Sql Elastic Pools.