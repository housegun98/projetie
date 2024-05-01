 "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "subscriptionId": {
            "type": "string"
        },
        "name": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "hostingPlanName": {
            "type": "string"
        },
        "serverFarmResourceGroup": {
            "type": "string"
        },
        "databaseName": {
            "type": "string"
        },
        "ftpsState": {
            "type": "string"
        },
        "sku": {
            "type": "string"
        },
        "skuCode": {
            "type": "string"
        },
        "workerSize": {
            "type": "string"
        },
        "workerSizeId": {
            "type": "string"
        },
        "numberOfWorkers": {
            "type": "string"
        },
        "linuxFxVersion": {
            "type": "string"
        },
        "storageSizeMB": {
            "type": "int"
        },
        "haEnabled": {
            "type": "string"
        },
        "availabilityZone": {
            "type": "string"
        },
        "backupRetentionDays": {
            "type": "int"
        },
        "geoRedundantBackup": {
            "type": "string"
        },
        "postgreServerSkuName": {
            "type": "string"
        },
        "serverEdition": {
            "type": "string"
        },
        "serverName": {
            "type": "string"
        },
        "serverUsername": {
            "type": "string"
        },
        "serverPassword": {
            "type": "securestring"
        },
        "publicNetworkAccess": {
            "type": "string"
        },
        "collation": {
            "type": "string"
        },
        "charset": {
            "type": "string"
        },
        "isVnetEnabled": {
            "type": "bool"
        },
        "isPrivateEndpointForAppEnabled": {
            "type": "bool"
        },
        "vnetName": {
            "type": "string"
        },
        "privateEndpointSubnet": {
            "type": "string"
        },
        "subnetForApp": {
            "type": "string"
        },
        "subnetForDb": {
            "type": "string"
        },
        "privateEndpointNameForApp": {
            "type": "string"
        },
        "privateEndpointNameForDb": {
            "type": "string"
        },
        "privateEndpointNameForCache": {
            "type": "string"
        },
        "privateDnsZoneNameForApp": {
            "type": "string"
        },
        "privateDnsZoneNameForDb": {
            "type": "string"
        },
        "privateDnsZoneNameForCache": {
            "type": "string"
        },
        "isCacheEnabled": {
            "type": "bool"
        },
        "cacheName": {
            "type": "string"
        }
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "[variables('resourcesDeploymentApiVersion')]",
            "name": "[variables('databaseResourcesDeployment')]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "condition": "[parameters('isVnetEnabled')]",
                            "type": "Microsoft.Resources/deployments",
                            "apiVersion": "[variables('resourcesDeploymentApiVersion')]",
                            "name": "[concat(variables('databaseResourcesDeployment'), '-vnet')]",
                            "properties": {
                                "mode": "Incremental",
                                "template": {
                                    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                                    "contentVersion": "1.0.0.0",
                                    "parameters": {},
                                    "variables": {},
                                    "resources": [
                                        {
                                            "type": "Microsoft.Network/virtualNetworks/subnets",
                                            "apiVersion": "[variables('vnetDeploymentApiVersion')]",
                                            "name": "[concat(parameters('vnetName'), '/', parameters('subnetForDb'))]",
                                            "properties": {
                                                "delegations": [
                                                    {
                                                        "name": "dlg-database",
                                                        "properties": {
                                                            "serviceName": "Microsoft.DBforPostgreSQL/flexibleServers"
                                                        }
                                                    }
                                                ],
                                                "serviceEndpoints": [],
                                                "addressPrefix": "[variables('subnetAddressForDb')]"
                                            }
                                        }
                                    ]
                                }
                            },
                            "dependsOn": []
                        },
                        {
                            "apiVersion": "[variables('databaseApiVersion')]",
                            "location": "[parameters('location')]",
                            "name": "[parameters('serverName')]",
                            "type": "Microsoft.DBforPostgreSQL/flexibleServers",
                            "tags": {
                                "datak": "",
                                "logistique": "",
                                "4.0": ""
                            },
                            "properties": {
                                "version": "[variables('databaseVersion')]",
                                "administratorLogin": "[parameters('serverUsername')]",
                                "administratorLoginPassword": "[parameters('serverPassword')]",
                                "publicNetworkAccess": "[parameters('publicNetworkAccess')]",
                                "haEnabled": "[parameters('haEnabled')]",
                                "storageProfile": {
                                    "storageMB": "[parameters('storageSizeMB')]",
                                    "backupRetentionDays": "[parameters('backupRetentionDays')]",
                                    "geoRedundantBackup": "[parameters('geoRedundantBackup')]"
                                },
                                "availabilityZone": "[parameters('availabilityZone')]",
                                "DelegatedSubnetArguments": {
                                    "SubnetArmResourceId": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('vnetName'), parameters('subnetForDb'))]"
                                },
                                "PrivateDnsZoneArguments": {
                                    "PrivateDnsZoneArmResourceId": "[resourceId('Microsoft.Network/privateDnsZones', parameters('privateDnsZoneNameForDb'))]"
                                }
                            },
                            "sku": {
                                "name": "[parameters('postgreServerSkuName')]",
                                "tier": "[parameters('serverEdition')]"
                            },
                            "dependsOn": [
                                "[concat(variables('databaseResourcesDeployment'), '-vnet')]"
                            ],
                            "resources": [
                                {
                                    "dependsOn": [
                                        "[concat('Microsoft.DBforPostgreSQL/flexibleServers', '/', parameters('serverName'))]"
                                    ],
                                    "type": "databases",
                                    "apiVersion": "2020-11-05-preview",
                                    "name": "[parameters('databaseName')]",
                                    "properties": {
                                        "charset": "[parameters('charset')]",
                                        "collation": "[parameters('collation')]"
                                    }
                                }
                            ]
                        }
                    ]
                }
            },
            "dependsOn": [
                "[concat('Microsoft.Resources/deployments', '/', variables('vnetResourcesDeployment'))]"
            ]
        },
        {
            "condition": "[parameters('isCacheEnabled')]",
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "[variables('resourcesDeploymentApiVersion')]",
            "name": "[variables('cacheResourcesDeployment')]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "apiVersion": "[variables('privateEndpointApiVersion')]",
                            "name": "[parameters('privateEndpointNameForCache')]",
                            "location": "[parameters('location')]",
                            "type": "Microsoft.Network/privateEndpoints",
                            "properties": {
                                "subnet": {
                                    "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('vnetName'), parameters('privateEndpointSubnet'))]"
                                },
                                "privateLinkServiceConnections": [
                                    {
                                        "name": "[parameters('privateEndpointNameForCache')]",
                                        "properties": {
                                            "privateLinkServiceId": "[resourceId('Microsoft.Cache/Redis/', parameters('cacheName'))]",
                                            "groupIds": [
                                                "redisCache"
                                            ]
                                        }
                                    }
                                ]
                            },
                            "resources": [
                                {
                                    "type": "privateDnsZoneGroups",
                                    "apiVersion": "[variables('privateEndpointApiVersion')]",
                                    "name": "default",
                                    "location": "[parameters('location')]",
                                    "properties": {
                                        "privateDnsZoneConfigs": [
                                            {
                                                "name": "privatelink-redis-cache-windows-net",
                                                "properties": {
                                                    "privateDnsZoneId": "[resourceId('Microsoft.Network/privateDnsZones', parameters('privateDnsZoneNameForCache'))]"
                                                }
                                            }
                                        ]
                                    },
                                    "dependsOn": [
                                        "[resourceId('Microsoft.Network/privateEndpoints', parameters('privateEndpointNameForCache'))]"
                                    ]
                                }
                            ],
                            "dependsOn": [
                                "[concat('Microsoft.Cache/Redis/', parameters('cacheName'))]"
                            ]
                        },
                        {
                            "name": "[parameters('cacheName')]",
                            "type": "Microsoft.Cache/Redis",
                            "location": "[parameters('location')]",
                            "apiVersion": "[variables('cacheApiVersion')]",
                            "tags": {},
                            "properties": {
                                "sku": {
                                    "name": "Standard",
                                    "family": "C",
                                    "capacity": "1"
                                },
                                "redisConfiguration": {},
                                "enableNonSslPort": false,
                                "redisVersion": "6",
                                "publicNetworkAccess": "Disabled"
                            }
                        }
                    ]
                }
            },
            "dependsOn": [
                "[concat('Microsoft.Resources/deployments', '/', variables('vnetResourcesDeployment'))]"
            ]
        },
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "[variables('resourcesDeploymentApiVersion')]",
            "name": "[variables('appServiceResourcesDeployment')]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "apiVersion": "[variables('appServicesApiVersion')]",
                            "name": "[parameters('hostingPlanName')]",
                            "type": "Microsoft.Web/serverfarms",
                            "location": "[parameters('location')]",
                            "kind": "linux",
                            "tags": {
                                "datak": "",
                                "logistique": "",
                                "4.0": ""
                            },
                            "properties": {
                                "name": "[parameters('hostingPlanName')]",
                                "workerSize": "[parameters('workerSize')]",
                                "workerSizeId": "[parameters('workerSizeId')]",
                                "numberOfWorkers": "[parameters('numberOfWorkers')]",
                                "reserved": true
                            },
                            "sku": {
                                "Tier": "[parameters('sku')]",
                                "Name": "[parameters('skuCode')]"
                            }
                        },
                        {
                            "apiVersion": "[variables('appServicesApiVersion')]",
                            "name": "[parameters('name')]",
                            "type": "Microsoft.Web/sites",
                            "location": "[parameters('location')]",
                            "tags": {
                                "datak": "",
                                "logistique": "",
                                "4.0": ""
                            },
                            "dependsOn": [
                                "[concat('Microsoft.Web/serverfarms/', parameters('hostingPlanName'))]"
                            ],
                            "properties": {
                                "name": "[parameters('name')]",
                                "siteConfig": {
                                    "linuxFxVersion": "[parameters('linuxFxVersion')]",
                                    "appSettings": [],
                                    "vnetRouteAllEnabled": true,
                                    "ftpsState": "[parameters('ftpsState')]"
                                },
                                "serverFarmId": "[concat('/subscriptions/', parameters('subscriptionId'),'/resourcegroups/', parameters('serverFarmResourceGroup'), '/providers/Microsoft.Web/serverfarms/', parameters('hostingPlanName'))]",
                                "clientAffinityEnabled": false
                            }
                        },
                        {
                            "condition": "[parameters('isVnetEnabled')]",
                            "type": "Microsoft.Resources/deployments",
                            "apiVersion": "[variables('resourcesDeploymentApiVersion')]",
                            "name": "[concat(variables('appServiceResourcesDeployment'), '-vnet')]",
                            "properties": {
                                "mode": "Incremental",
                                "template": {
                                    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                                    "contentVersion": "1.0.0.0",
                                    "parameters": {},
                                    "variables": {},
                                    "resources": [
                                        {
                                            "type": "Microsoft.Network/virtualNetworks/subnets",
                                            "apiVersion": "[variables('vnetDeploymentApiVersion')]",
                                            "name": "[concat(parameters('vnetName'), '/', parameters('subnetForApp'))]",
                                            "properties": {
                                                "delegations": [
                                                    {
                                                        "name": "dlg-appServices",
                                                        "properties": {
                                                            "serviceName": "Microsoft.Web/serverfarms"
                                                        }
                                                    }
                                                ],
                                                "serviceEndpoints": [],
                                                "addressPrefix": "[variables('subnetAddressForApp')]"
                                            }
                                        },
                                        {
                                            "apiVersion": "[variables('appServicesApiVersion')]",
                                            "name": "[format('{0}/{1}', parameters('name'), 'virtualNetwork')]",
                                            "type": "Microsoft.Web/sites/networkConfig",
                                            "dependsOn": [
                                                "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('vnetName'), parameters('subnetForApp'))]"
                                            ],
                                            "properties": {
                                                "subnetResourceId": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('vnetName'), parameters('subnetForApp'))]"
                                            }
                                        },
                                        {
                                            "condition": "[parameters('isPrivateEndpointForAppEnabled')]",
                                            "apiVersion": "[variables('privateEndpointApiVersion')]",
                                            "name": "[parameters('privateEndpointNameForApp')]",
                                            "location": "[parameters('location')]",
                                            "type": "Microsoft.Network/privateEndpoints",
                                            "properties": {
                                                "subnet": {
                                                    "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('vnetName'), parameters('privateEndpointSubnet'))]"
                                                },
                                                "privateLinkServiceConnections": [
                                                    {
                                                        "name": "[parameters('privateEndpointNameForApp')]",
                                                        "properties": {
                                                            "privateLinkServiceId": "[resourceId('Microsoft.Web/Sites', parameters('name'))]",
                                                            "groupIds": [
                                                                "sites"
                                                            ]
                                                        }
                                                    }
                                                ]
                                            },
                                            "resources": [
                                                {
                                                    "condition": "[parameters('isPrivateEndpointForAppEnabled')]",
                                                    "type": "privateDnsZoneGroups",
                                                    "apiVersion": "[variables('privateEndpointApiVersion')]",
                                                    "name": "default",
                                                    "location": "[parameters('location')]",
                                                    "properties": {
                                                        "privateDnsZoneConfigs": [
                                                            {
                                                                "name": "[parameters('privateDnsZoneNameForApp')]",
                                                                "properties": {
                                                                    "privateDnsZoneId": "[resourceId('Microsoft.Network/privateDnsZones', parameters('privateDnsZoneNameForApp'))]"
                                                                }
                                                            }
                                                        ]
                                                    },
                                                    "dependsOn": [
                                                        "[resourceId('Microsoft.Network/privateEndpoints', parameters('privateEndpointNameForApp'))]"
                                                    ]
                                                }
                                            ]
                                        }
                                    ]
                                }
                            },
                            "dependsOn": [
                                "[resourceId('Microsoft.Web/serverfarms', parameters('hostingPlanName'))]",
                                "[resourceId('Microsoft.Web/sites', parameters('name'))]"
                            ]
                        }
                    ]
                }
            },
            "dependsOn": [
                "[concat('Microsoft.Resources/deployments', '/', variables('vnetResourcesDeployment'))]"
            ]
        },
        {
            "condition": "[parameters('isVnetEnabled')]",
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "[variables('resourcesDeploymentApiVersion')]",
            "name": "[variables('vnetResourcesDeployment')]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "type": "Microsoft.Network/virtualNetworks",
                            "apiVersion": "[variables('vnetDeploymentApiVersion')]",
                            "location": "[parameters('location')]",
                            "name": "[parameters('vnetName')]",
                            "properties": {
                                "addressSpace": {
                                    "addressPrefixes": [
                                        "[variables('vnetAddress')]"
                                    ]
                                },
                                "subnets": []
                            },
                            "resources": [
                                {
                                    "type": "subnets",
                                    "apiVersion": "[variables('vnetDeploymentApiVersion')]",
                                    "name": "[parameters('privateEndpointSubnet')]",
                                    "properties": {
                                        "delegations": [],
                                        "serviceEndpoints": [],
                                        "addressPrefix": "[variables('subnetAddressForPrivateEndpoint')]",
                                        "privateEndpointNetworkPolicies": "Disabled"
                                    },
                                    "dependsOn": [
                                        "[resourceId('Microsoft.Network/virtualNetworks', parameters('vnetName'))]"
                                    ]
                                }
                            ]
                        },
                        {
                            "condition": "[parameters('isPrivateEndpointForAppEnabled')]",
                            "type": "Microsoft.Network/privateDnsZones",
                            "apiVersion": "[variables('privateDnsApiVersion')]",
                            "name": "[parameters('privateDnsZoneNameForApp')]",
                            "location": "global",
                            "resources": [
                                {
                                    "condition": "[parameters('isPrivateEndpointForAppEnabled')]",
                                    "type": "virtualNetworkLinks",
                                    "apiVersion": "[variables('privateDnsApiVersion')]",
                                    "name": "[format('{0}-applink', parameters('privateDnsZoneNameForApp'))]",
                                    "location": "global",
                                    "dependsOn": [
                                        "[resourceId('Microsoft.Network/privateDnsZones', parameters('privateDnsZoneNameForApp'))]"
                                    ],
                                    "properties": {
                                        "virtualNetwork": {
                                            "id": "[resourceId('Microsoft.Network/virtualNetworks', parameters('vnetName'))]"
                                        },
                                        "registrationEnabled": false
                                    }
                                }
                            ]
                        },
                        {
                            "type": "Microsoft.Network/privateDnsZones",
                            "apiVersion": "[variables('privateDnsApiVersion')]",
                            "name": "[parameters('privateDnsZoneNameForDb')]",
                            "location": "global",
                            "resources": [
                                {
                                    "type": "virtualNetworkLinks",
                                    "apiVersion": "[variables('privateDnsApiVersion')]",
                                    "name": "[format('{0}-dblink', parameters('privateDnsZoneNameForDb'))]",
                                    "location": "global",
                                    "dependsOn": [
                                        "[resourceId('Microsoft.Network/privateDnsZones', parameters('privateDnsZoneNameForDb'))]"
                                    ],
                                    "properties": {
                                        "virtualNetwork": {
                                            "id": "[resourceId('Microsoft.Network/virtualNetworks', parameters('vnetName'))]"
                                        },
                                        "registrationEnabled": false
                                    }
                                }
                            ]
                        },
                        {
                            "type": "Microsoft.Network/privateDnsZones",
                            "apiVersion": "[variables('privateDnsApiVersion')]",
                            "name": "[parameters('privateDnsZoneNameForCache')]",
                            "location": "global",
                            "resources": [
                                {
                                    "type": "virtualNetworkLinks",
                                    "apiVersion": "[variables('privateDnsApiVersion')]",
                                    "name": "[format('{0}-applink', parameters('privateDnsZoneNameForCache'))]",
                                    "location": "global",
                                    "dependsOn": [
                                        "[resourceId('Microsoft.Network/privateDnsZones', parameters('privateDnsZoneNameForCache'))]"
                                    ],
                                    "properties": {
                                        "virtualNetwork": {
                                            "id": "[resourceId('Microsoft.Network/virtualNetworks', parameters('vnetName'))]"
                                        },
                                        "registrationEnabled": false
                                    }
                                }
                            ]
                        }
                    ]
                }
            }
        },
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "[variables('resourcesDeploymentApiVersion')]",
            "name": "[variables('appServiceDatabaseConnectionResourcesDeployment')]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "resources": [
                        {
                            "apiVersion": "[variables('serviceConnectorApiVersion')]",
                            "name": "[variables('serviceConnectorName')]",
                            "type": "Microsoft.ServiceLinker/linkers",
                            "scope": "[resourceId('Microsoft.Web/sites', parameters('name'))]",
                            "properties": {
                                "targetService": {
                                    "type": "AzureResource",
                                    "id": "[resourceId('Microsoft.DBForPostgreSQL/flexibleServers/databases', parameters('serverName'), parameters('databaseName'))]"
                                },
                                "authInfo": {
                                    "authType": "secret",
                                    "name": "[parameters('serverUsername')]",
                                    "secretInfo": {
                                        "secretType": "rawValue",
                                        "value": "[parameters('serverPassword')]"
                                    }
                                },
                                "clientType": "java",
                                "vNetSolution": null
                            }
                        }
                    ]
                }
            },
            "dependsOn": [
                "[concat('Microsoft.Resources/deployments', '/', variables('databaseResourcesDeployment'))]",
                "[concat('Microsoft.Resources/deployments', '/', variables('appServiceResourcesDeployment'))]"
            ]
        }
    ],
    "variables": {
        "vnetResourcesDeployment": "vnetResourcesDeployment",
        "databaseResourcesDeployment": "databaseResourcesDeployment",
        "cacheResourcesDeployment": "cacheResourcesDeployment",
        "appServiceResourcesDeployment": "appServiceResourcesDeployment",
        "appServiceDatabaseConnectionResourcesDeployment": "appServiceDatabaseConnectionResourcesDeployment",
        "resourcesDeploymentApiVersion": "2020-06-01",
        "appServicesApiVersion": "2018-11-01",
        "databaseApiVersion": "2020-02-14-preview",
        "databaseVersion": "14",
        "vnetDeploymentApiVersion": "2020-07-01",
        "privateDnsApiVersion": "2018-09-01",
        "privateEndpointApiVersion": "2021-02-01",
        "vnetAddress": "10.0.0.0/16",
        "subnetAddressForPrivateEndpoint": "10.0.0.0/24",
        "subnetAddressForApp": "10.0.1.0/24",
        "subnetAddressForDb": "10.0.2.0/24",
        "cacheApiVersion": "2022-06-01",
        "serviceConnectorApiVersion": "2022-05-01",
        "serviceConnectorName": "defaultConnector",
        "serviceConnectorRedisName": "RedisConnector"
    }
}
