---
title: "Associate Azure Public IP to an existing network interface using Powershell"
description: "How to associate a Public IP address in Azure to an existing network interface using Powershell."
summary: "How to associate a Public IP address in Azure to an existing network interface using Powershell."
date: 2020-03-09
draft: false
author: richard
thumbnail: /posts/public-ip.png
featureImage: /posts/public-ip.png

tags: [
    "azure",
    "arm templates"
]
---
Today I needed to associate a public IP address to an existing Azure virtual machine. The Microsoft Docs website has [an article](https://docs.microsoft.com/en-us/azure/virtual-network/associate-public-ip-address-vm) that describes how to do it. But somehow [the Powershell example](https://docs.microsoft.com/en-us/azure/virtual-network/associate-public-ip-address-vm#powershell) didn't work. 

In this short post I will provide you with a Powershell snippet that will actually work.

## The Powershell example

The example below creates a public IP address in Azure and associates it to an existing Network Interface. 

* You can either use the [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview) or use a local installation of Powershell to execute the code snippet. If you are using Powershell locally, make sure you have the [Azure Powershell module](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps) installed. 
* If you have multiple subscriptions, use the `Select-AzSubscription` command first to select the right subscription. 

```powershell
    $pip = New-AzPublicIpAddress -Name 'MyPublicIpName' -ResourceGroupName 'MyResourceGroupName' -Sku 'Basic' -AllocationMethod 'Dynamic' -Location 'West Europe'
    $vnet = Get-AzVirtualNetwork -Name 'MyVnetName' -ResourceGroupName 'MyResourceGroupName'
    $subnet = Get-AzVirtualNetworkSubnetConfig -Name 'MySubnetName' -VirtualNetwork $vnet
    $nic = Get-AzNetworkInterface -Name 'MyNicName' -ResourceGroupName 'MyResourceGroupName'
    $ipconfig = New-AzNetworkInterfaceIpConfig -Name 'MyIpConfigName' -Subnet $subnet -PublicIpAddress $pip
    $nic | Add-AzNetworkInterfaceIpConfig -Name $ipconfigName -Subnet $subnet -PublicIpAddress $pip
    $nic | Set-AzNetworkInterface
```

In the example I create a public IP of Sku "Basic", allocation method "Dynamic" and in region "West Europe". You might need to change these values to your specific needs. Consult the docs on the New-AzPublicIpAddress cmdlet [here](https://docs.microsoft.com/en-us/powershell/module/az.network/new-azpublicipaddress).

## Conclusion

If Powershell is your tool of choice, and you need add a public IP address to an existing network interface, this code snippet will work for you. 
