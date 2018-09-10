"# Azure-azure-quickstart-templates-master-101-Multi-VM-Deployment-microsoft.compute-" 
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "_artifactsLocation": {
        "type": "string",
        "defaultValue": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-VM-Virus-Attack-Prevention",
        "metadata": {
          "description": "this will be the location for artifacts"
        }
      },
      "_artifactsLocationSasToken": {
        "type": "securestring",
        "defaultValue": "",
        "metadata": {
          "description": "this will be the sas key to access artifacts"
        }
      },
      "location": {
        "type": "string",
        "allowedValues": [
          "East US",
          "West Europe",
          "Southeast Asia",
          "Australia Southeast"
        ],
        "defaultValue": "East US",
        "metadata": {
          "description": "your resources will be created in this location"
        }
      },
      "omsSku": {
        "type": "string",
        "defaultValue": "PerNode",
        "allowedValues": [
          "Free",
          "Standalone",
          "PerNode"
        ],
        "metadata": {
          "description": "this will be you SKU for OMS"
        }
      },
      "vNetName": {
        "type": "string",
        "defaultValue": "virus-attack-vnet",
        "metadata": {
          "description": "this will be name of the VNET"
        }
      },
      "vNetAddressSpace": {
        "type": "string",
        "defaultValue": "10.1.0.0/16",
        "metadata": {
          "description": "this will be address space for VNET"
        }
      },
      "subnetName": {
        "type": "string",
        "defaultValue": "subnet-01",
        "metadata": {
          "description": "this will be name of the subnet"
        }
      },
      "subnetAddressSpace": {
        "type": "string",
        "defaultValue": "10.1.0.0/24",
        "metadata": {
          "description": "this will be address space for subnet"
        }
      },
      "nsgNamePrefix": {
        "type": "string",
        "defaultValue": "virus-attack",
        "metadata": {
          "description": "this will be prefix used for NSG"
        }
      },
      "pipAddressType": {
        "type": "string",
        "defaultValue": "dynamic",
        "allowedValues": [
          "dynamic",
          "static"
        ],
        "metadata": {
          "description": "this will be the type of public IP address used for the VM name"
        }
      },
      "vmNamePrefix": {
        "type": "string",
        "defaultValue": "vm",
        "metadata": {
          "description": "this will be the prefix used for the VM name"
        }
      },
      "adminUserName": {
        "type": "string",
        "metadata": {
          "description": "this will be the user name of the VMs deployed"
        }
      },
      "adminUserPassword": {
        "type": "securestring",
        "metadata": {
          "description": "this will be the password of the user created on the user"
        }
      }
    },
    "variables": {
      "omsWorkspaceName": "[concat('Virus-Attack','-',substring(uniqueString(resourceGroup().id),0,5))]",
      "omsSolutions": [
        "Security",
        "AzureActivity",
        "AntiMalware"
      ],
      "tags": {
        "scenario": "Vm-Virus-Attack-Prevention"
      },
      "subnets": [
        {
          "name": "[parameters('subnetName')]",
          "properties": {
            "addressPrefix": "[parameters('subnetAddressSpace')]"
          }
        }
      ],
      "nsgName": "[concat(parameters('nsgNamePrefix'),'-','nsg')]",
      "nsgSecurityRules": [
        {
          "name": "allow-inbound-rdp",
          "properties": {
            "protocol": "Tcp",
            "sourcePortRange": "*",
            "destinationPortRange": "3389",
            "sourceAddressPrefix": "*",
            "destinationAddressPrefix": "*",
            "access": "Allow",
            "priority": 1000,
            "direction": "Inbound"
          }
        }
      ],
      "vmNames": [
        "[concat(parameters('vmNamePrefix'),'-without-ep')]",
        "[concat(parameters('vmNamePrefix'),'-with-ep')]"
      ],
      "omsTemplateUri": "[concat(parameters('_artifactsLocation'),'/nested/microsoft.loganalytics/workspaces.json',parameters('_artifactsLocationSasToken'))]",
      "vnetTemplateUri": "[concat(parameters('_artifactsLocation'),'/nested/microsoft.network/virtualnetworks.json',parameters('_artifactsLocationSasToken'))]",
      "nsgTemplateUri": "[concat(parameters('_artifactsLocation'),'/nested/microsoft.network/nsg.json',parameters('_artifactsLocationSasToken'))]",
      "pipTemplateUri": "[concat(parameters('_artifactsLocation'),'/nested/microsoft.network/publicipaddress.json',parameters('_artifactsLocationSasToken'))]",
      "nicTemplateUri": "[concat(parameters('_artifactsLocation'),'/nested/microsoft.network/nic-with-pip.json',parameters('_artifactsLocationSasToken'))]",
      "vmTemplateUri": "[concat(parameters('_artifactsLocation'),'/nested/microsoft.compute/vm.windows.json',parameters('_artifactsLocationSasToken'))]"
    },
    "resources": [
      {
        "apiVersion": "2017-05-10",
        "name": "deploy-virus-attack-oms-resource",
        "type": "Microsoft.Resources/deployments",
        "properties": {
          "mode": "Incremental",
          "templateLink": {
            "uri": "[variables('omsTemplateUri')]"
          },
          "parameters": {
            "omsWorkspaceName": {
              "value": "[variables('omsWorkspaceName')]"
            },
            "omsSolutionsName": {
              "value": "[variables('omsSolutions')]"
            },
            "sku": {
              "value": "[parameters('omsSku')]"
            },
            "location": {
              "value": "[parameters('location')]"
            },
            "tags": {
              "value": "[variables('tags')]"
            }
          }
        }
      },
      {
        "apiVersion": "2017-05-10",
        "name": "[concat(parameters('vnetName'),'-','-resource')]",
        "type": "Microsoft.Resources/deployments",
        "properties": {
          "mode": "Incremental",
          "templateLink": {
            "uri": "[variables('vnetTemplateUri')]"
          },
          "parameters": {
            "vnetName": {
              "value": "[parameters('vnetName')]"
            },
            "addressPrefix": {
              "value": "[parameters('vNetAddressSpace')]"
            },
            "subnets": {
              "value": "[variables('subnets')]"
            },
            "location": {
              "value": "[parameters('location')]"
            },
            "tags": {
              "value": "[variables('tags')]"
            }
          }
        }
      },
      {
        "apiVersion": "2017-05-10",
        "name": "[concat(variables('nsgName'),'-','-resource')]",
        "type": "Microsoft.Resources/deployments",
        "properties": {
          "mode": "Incremental",
          "templateLink": {
            "uri": "[variables('nsgTemplateUri')]"
          },
          "parameters": {
            "nsgName": {
              "value": "[variables('nsgName')]"
            },
            "securityRules": {
              "value": "[variables('nsgSecurityRules')]"
            },
            "location": {
              "value": "[parameters('location')]"
            },
            "tags": {
              "value": "[variables('tags')]"
            }
          }
        }
      },
      {
        "apiVersion": "2017-05-10",
        "name": "[concat(variables('vmNames')[copyIndex()],'-pip','-resource')]",
        "type": "Microsoft.Resources/deployments",
        "copy": {
          "name": "copy-pip",
          "count": 2
        },
        "properties": {
          "mode": "Incremental",
          "templateLink": {
            "uri": "[variables('pipTemplateUri')]"
          },
          "parameters": {
            "publicIPAddressName": {
              "value": "[concat(variables('vmNames')[copyIndex()],'-pip')]"
            },
            "publicIPAddressType": {
              "value": "[parameters('pipAddressType')]"
            },
            "dnsNameForPublicIP": {
              "value": "[concat(variables('vmNames')[copyIndex()],uniqueString(resourceGroup().id, 'pip'),'-pip')]"
            },
            "location": {
              "value": "[parameters('location')]"
            },
            "tags": {
              "value": "[variables('tags')]"
            }
          }
        }
      },
      {
        "apiVersion": "2017-05-10",
        "name": "[concat(variables('vmNames')[copyIndex()],'-nic','-resource')]",
        "type": "Microsoft.Resources/deployments",
        "dependsOn": [
          "copy-pip",
          "[concat(variables('nsgName'),'-','-resource')]",
          "[concat(parameters('vnetName'),'-','-resource')]"
        ],
        "copy": {
          "name": "copy-nic",
          "count": 2
        },
        "properties": {
          "mode": "Incremental",
          "templateLink": {
            "uri": "[variables('nicTemplateUri')]"
          },
          "parameters": {
            "nicName": {
              "value": "[concat(variables('vmNames')[copyIndex()],'-nic')]"
            },
            "publicIPAddressId": {
              "value": "[resourceId('Microsoft.Network/publicIPAddresses',concat(variables('vmNames')[copyIndex()],'-pip'))]"
            },
            "subnetId": {
              "value": "[resourceId('Microsoft.Network/virtualNetworks/subnets',parameters('vnetName'), variables('subnets')[0].name)]"
            },
            "location": {
              "value": "[parameters('location')]"
            },
            "nsgId": {
              "value": "[resourceId('Microsoft.Network/networkSecurityGroups',variables('nsgName'))]"
            },
            "tags": {
              "value": "[variables('tags')]"
            }
          }
        }
      },
      {
        "apiVersion": "2017-05-10",
        "name": "[concat(variables('vmNames')[copyIndex()])]",
        "type": "Microsoft.Resources/deployments",
        "dependsOn": [
          "copy-nic"
        ],
        "copy": {
          "name": "copy-vm",
          "count": 2
        },
        "properties": {
          "mode": "Incremental",
          "templateLink": {
            "uri": "[variables('vmTemplateUri')]"
          },
          "parameters": {
            "vmName": {
              "value": "[concat(variables('vmNames')[copyIndex()])]"
            },
            "adminUsername": {
              "value": "[parameters('adminUserName')]"
            },
            "adminPassword": {
              "value": "[parameters('adminUserPassword')]"
            },
            "nicId": {
              "value": "[resourceId('Microsoft.Network/networkInterfaces',concat(variables('vmNames')[copyIndex()],'-nic'))]"
            },
            "location": {
              "value": "[parameters('location')]"
            },
            "tags": {
              "value": "[variables('tags')]"
            }
          }
        }
      },
      {
        "type": "Microsoft.Compute/virtualMachines/extensions",
        "name": "[concat(concat(variables('vmNames')[copyIndex()]),'/OMSExtension')]",
        "apiVersion": "2017-12-01",
        "location": "[parameters('location')]",
        "dependsOn": [
          "[concat(variables('vmNames')[copyIndex()])]",
          "deploy-virus-attack-oms-resource"
        ],
        "copy": {
          "name": "copy-monitoring-agent",
          "count": 2
        },
        "properties": {
          "publisher": "Microsoft.EnterpriseCloud.Monitoring",
          "type": "MicrosoftMonitoringAgent",
          "typeHandlerVersion": "1.0",
          "autoUpgradeMinorVersion": true,
          "settings": {
            "workspaceId": "[reference('deploy-virus-attack-oms-resource').outputs.workspaceId.value]"
          },
          "protectedSettings": {
            "workspaceKey": "[reference('deploy-virus-attack-oms-resource').outputs.workspaceKey.value]"
          }
        }
      },
      {
        "type": "Microsoft.Compute/virtualMachines/extensions",
        "name": "[concat(variables('vmNames')[copyIndex(1)],'/malware')]",
        "apiVersion": "2017-12-01",
        "location": "[parameters('location')]",
        "dependsOn": [
          "copy-monitoring-agent"
        ],
        "copy": {
          "name": "copy-extension-malware",
          "count": 1
        },
        "properties": {
          "publisher": "Microsoft.Azure.Security",
          "type": "IaaSAntimalware",
          "typeHandlerVersion": "1.1",
          "autoUpgradeMinorVersion": true,
          "settings": {
            "AntimalwareEnabled": "true",
            "Exclusions": {
              "Paths": "",
              "Extensions": "",
              "Processes": "taskmgr.exe"
            },
            "RealtimeProtectionEnabled": "true",
            "ScheduledScanSettings": {
              "isEnabled": "true",
              "scanType": "Quick",
              "day": "7",
              "time": "120"
            }
          },
          "protectedSettings": null
        }
      }
    ],
    "outputs": {}
  }
