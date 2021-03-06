<properties
	pageTitle="Escalado automático de conjuntos de escalado de máquinas virtuales Linux | Microsoft Azure"
	description="Configuración del escalado automático para un conjunto de escalado de máquinas virtuales Linux mediante la CLI de Azure"
	services="virtual-machine-scale-sets"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machine-scale-sets"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="06/10/2016"
	ms.author="davidmu"/>

# Escalado automático de máquinas de Linux en un conjunto de escalado de máquinas virtuales

Los conjuntos de escala de máquinas virtuales facilitan la implementación y administración de máquinas virtuales idénticas como un conjunto. Los conjuntos de escala proporcionan una capa de proceso altamente escalable y personalizable para aplicaciones de gran escala y admiten imágenes de la plataforma Windows, imágenes de la plataforma Linux, imágenes personalizadas y extensiones. Para obtener más información, consulte [Virtual Machine Scale Sets Overview (Información general de conjuntos de escalado de máquinas virtuales)](virtual-machine-scale-sets-overview.md).

En este tutorial se muestra cómo crear un conjunto de escalado de máquinas virtuales de máquinas virtuales de Linux usando la versión más reciente de Ubuntu Linux y escalar automáticamente las máquinas en el conjunto. Esto se hace creando una plantilla de Azure Resource Manager e implementándola mediante la CLI de Azure. Para más información sobre las plantillas, consulte [Creación de plantillas del Administrador de recursos de Azure](../resource-group-authoring-templates.md). Para obtener más información sobre el escalado automático de conjuntos de escalado, consulte [Automatic scaling and Virtual Machine Scale Sets (Escalado automático y conjuntos de escalado de máquinas virtuales)](virtual-machine-scale-sets-autoscale-overview.md).

En este tutorial, implemente los recursos y las extensiones siguientes:

- Microsoft.Storage/storageAccounts
- Microsoft.Network/virtualNetworks
- Microsoft.Network/publicIPAddresses
- Microsoft.Network/loadBalancers
- Microsoft.Network/networkInterfaces
- Microsoft.Compute/virtualMachines
- Microsoft.Compute/virtualMachineScaleSets
- Microsoft.Insights.VMDiagnosticsSettings
- Microsoft.Insights/autoscaleSettings

Para más información sobre los recursos del Administrador de recursos, consulte [Proveedores de procesos, redes y almacenamiento de Azure en el Administrador de recursos de Azure](../virtual-machines/virtual-machines-linux-compare-deployment-models.md).

Antes de empezar con los pasos de este tutorial, [instale la CLI de Azure](../xplat-cli-install.md).

## Paso 1: Creación de un grupo de recursos y una cuenta de almacenamiento

1. **Iniciar sesión en Microsoft Azure**: en la interfaz de la línea de comandos (Bash, Terminal o símbolo del sistema), asegúrese de que esté en modo Resource Manager escribiendo `azure config mode arm` y, luego, [inicie sesión con su identificador profesional o educativo](../xplat-cli-connect.md#use-the-log-in-method) escribiendo `azure login` y siguiendo las indicaciones para disfrutar de una experiencia de inicio de sesión interactiva en su cuenta de Azure.

	> [AZURE.NOTE] Si tiene un identificador profesional o educativo y sabe que no tiene habilitada la autenticación en dos fases, puede usar `azure login -u` junto con el identificador profesional o educativo para iniciar una sesión sin una sesión interactiva. Si no tiene un identificador profesional o educativo, puede [crear uno desde su cuenta personal de Microsoft](../virtual-machines/resource-group-create-work-id-from-personal.md).

2. **Cree un grupo de recursos**: todos los recursos deben implementarse en un grupo de recursos. Para este tutorial, asigne el nombre **vmsstest1** al grupo de recursos.

        azure group create vmsstestrg1 centralus

3. **Implemente una cuenta de almacenamiento en el nuevo grupo de recursos**: este tutorial usa varias cuentas de almacenamiento para facilitar el conjunto de escala de máquinas virtuales. Cree una cuenta de almacenamiento denominada "**vmsstestsa**". Mantenga abierta la ventana de la interfaz de comandos para los pasos posteriores de este tutorial:

        azure storage account create -g vmsstestrg1 -l centralus --kind Storage --sku-name LRS vmsstestsa

## Paso 2: Creación de la plantilla
Las plantillas del Administrador de recursos de Azure le permiten implementar y administrar los distintos recursos de Azure en conjunto mediante una descripción de JSON de los recursos y los parámetros de implementación asociados.

1. En el editor que prefiera, cree el archivo C:\\VMSSTemplate.json y agregue la estructura inicial de JSON para admitir la plantilla.

        {
          "$schema":"http://schema.management.azure.com/schemas/2014-04-01-preview/VM.json",
          "contentVersion": "1.0.0.0",
          "parameters": {
          }
          "variables": {
          }
          "resources": [
          ]
        }

2. Los parámetros no siempre son necesarios, pero facilitan la administración de la plantilla. Proporcionan una manera de especificar valores para la plantilla, describen el tipo del valor, el valor predeterminado, en caso de ser necesario, y, posiblemente, los valores permitidos del parámetro. Agregue estos parámetros en el elemento primario de los parámetros que agregó a la plantilla.

        "vmName": {
          "type": "string"
        },
        "vmSSName": {
          "type": "string"
        },
        "instanceCount": {
          "type": "string"
        },
        "adminUsername": {
          "type": "string"
        },
        "adminPassword": {
          "type": "securestring"
        },
        "resourcePrefix": {
          "type": "string"
        }
            
	- Un nombre para la máquina virtual independiente que se utiliza para tener acceso a las máquinas del conjunto de escalado.
	- Un nombre para la cuenta de almacenamiento donde se almacena la plantilla.
	- El número de instancias de máquinas virtuales para crear inicialmente en el conjunto de escala.
	- Un nombre y contraseña de la cuenta de administrador en las máquinas virtuales.
	- Un prefijo para los recursos que se crean en el grupo de recursos.

3. Pueden usarse variables en una plantilla para especificar los valores que pueden cambiar con frecuencia o los valores que se deben crear a partir de una combinación de valores de parámetro. Agregue estas variables en el elemento primario de las variables que agregó a la plantilla.

        "dnsName1": "[concat(parameters('resourcePrefix'),'dn1')]",
        "dnsName2": "[concat(parameters('resourcePrefix'),'dn2')]",
        "vmSize": "Standard_A0",
        "imagePublisher": "Canonical",
        "imageOffer": "UbuntuServer",
        "imageVersion": "15.10",
        "addressPrefix": "10.0.0.0/16",
        "subnetName": "Subnet",
        "subnetPrefix": "10.0.0.0/24",
        "publicIP1": "[concat(parameters('resourcePrefix'),'ip1')]",
        "publicIP2": "[concat(parameters('resourcePrefix'),'ip2')]",
        "loadBalancerName": "[concat(parameters('resourcePrefix'),'lb1')]",
        "virtualNetworkName": "[concat(parameters('resourcePrefix'),'vn1')]",
        "nicName1": "[concat(parameters('resourcePrefix'),'nc1')]",
        "nicName2": "[concat(parameters('resourcePrefix'),'nc2')]",
        "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',variables('virtualNetworkName'))]",
        "publicIPAddressID1": "[resourceId('Microsoft.Network/publicIPAddresses',variables('publicIP1'))]",
        "publicIPAddressID2": "[resourceId('Microsoft.Network/publicIPAddresses',variables('publicIP2'))]",
        "lbID": "[resourceId('Microsoft.Network/loadBalancers',variables('loadBalancerName'))]",
        "nicId": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName2'))]",
        "frontEndIPConfigID": "[concat(variables('lbID'),'/frontendIPConfigurations/loadBalancerFrontEnd')]",
        "storageAccountType": "Standard_LRS",
        "storageAccountSuffix": [ "a", "g", "m", "s", "y" ],
        "diagnosticsStorageAccountName": "[concat(parameters('resourcePrefix'), 'a')]",
        "accountid": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/', resourceGroup().name,'/providers/','Microsoft.Storage/storageAccounts/', variables('diagnosticsStorageAccountName'))]",
        "wadlogs": "<WadCfg><DiagnosticMonitorConfiguration>",
        "wadperfcounter": "<PerformanceCounters scheduledTransferPeriod="PT1M"><PerformanceCounterConfiguration counterSpecifier="\\Processor\\PercentProcessorTime" sampleRate="PT15S" unit="Percent"><annotation displayName="CPU percentage guest OS" locale="es-ES"/></PerformanceCounterConfiguration></PerformanceCounters>",
        "wadcfgxstart": "[concat(variables('wadlogs'),variables('wadperfcounter'),'<Metrics resourceId="')]",
        "wadmetricsresourceid": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name ,'/providers/','Microsoft.Compute/virtualMachineScaleSets/',parameters('vmssName'))]",
        "wadcfgxend": "[concat('"><MetricAggregation scheduledTransferPeriod="PT1H"/><MetricAggregation scheduledTransferPeriod="PT1M"/></Metrics></DiagnosticMonitorConfiguration></WadCfg>')]"
        
	- Nombres DNS que usan las interfaces de red.
	- El tamaño de las máquinas virtuales que se usan en el conjunto de escala. Para más información sobre los tamaños de máquina virtual, consulte [Tamaños de máquina virtual](../virtual-machines/virtual-machines-size-specs.md).
	- La información de la imagen de plataforma para definir el sistema operativo que se ejecutará en las máquinas virtuales en el conjunto de escala. Para más información acerca de la selección de imágenes, consulte [Seleccione y navegue por imágenes de máquina virtual de Azure con PowerShell y la CLI de Azure](../virtual-machines/resource-groups-vm-searching.md).
	- Los nombres de direcciones IP y los prefijos para la red virtual y las subredes.
	- Los nombres y los identificadores de la red virtual, el equilibrador de carga y las interfaces de red.
	- Los nombres de cuentas de almacenamiento para las cuentas asociadas a las máquinas en el conjunto de escala.
	- Configuración de la extensión de diagnósticos que se instala en las máquinas virtuales. Para más información sobre la extensión de diagnósticos, consulte [Crear una máquina virtual de Windows con supervisión y diagnóstico mediante la plantilla del Administrador de recursos de Azure](../virtual-machines/virtual-machines-extensions-diagnostics-windows-template.md)

4. Agregue el recurso de la cuenta de almacenamiento en el elemento primario de recursos que agregó a la plantilla. Esta plantilla utiliza un bucle para crear las 5 cuentas de almacenamiento recomendadas donde se almacenan los discos del sistema operativo y los datos de diagnóstico. Este conjunto de cuentas puede admitir hasta 100 máquinas virtuales en un conjunto de escala, lo cual es el máximo actual. Cada cuenta de almacenamiento se denomina con un indicador de letra, que se definió en las variables, y se combina con el sufijo que proporcionó en los parámetros de la plantilla.

        {
          "type": "Microsoft.Storage/storageAccounts",
          "name": "[concat(parameters('resourcePrefix'), variables('storageAccountSuffix')[copyIndex()])]",
          "apiVersion": "2015-06-15",
          "copy": {
            "name": "storageLoop",
            "count": 5
          },
          "location": "[resourceGroup().location]",
          "properties": {
            "accountType": "[variables('storageAccountType')]"
          }
        },

5. Agregue el recurso de red virtual. Consulte [Proveedor de recursos de red](../virtual-network/resource-groups-networking.md) para más información.

        {
          "apiVersion": "2015-06-15",
          "type": "Microsoft.Network/virtualNetworks",
          "name": "[variables('virtualNetworkName')]",
          "location": "[resourceGroup().location]",
          "properties": {
            "addressSpace": {
              "addressPrefixes": [
                "[variables('addressPrefix')]"
              ]
            },
            "subnets": [
              {
                "name": "[variables('subnetName')]",
                "properties": {
                  "addressPrefix": "[variables('subnetPrefix')]"
                }
              }
            ]
          }
        },

6. Agregue los recursos de dirección IP públicos que usan la interfaz de red y el equilibrador de carga.

        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/publicIPAddresses",
          "name": "[variables('publicIP1')]",
          "location": "[resourceGroup().location]",
          "properties": {
            "publicIPAllocationMethod": "Dynamic",
            "dnsSettings": {
              "domainNameLabel": "[variables('dnsName1')]"
            }
          }
        },
        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/publicIPAddresses",
          "name": "[variables('publicIP2')]",
          "location": "[resourceGroup().location]",
          "properties": {
            "publicIPAllocationMethod": "Dynamic",
            "dnsSettings": {
              "domainNameLabel": "[variables('dnsName2')]"
            }
          }
        },

7. Agregue el recurso de equilibrador de carga que usa el conjunto de escala. Para más información, consulte [Compatibilidad del Administrador de recursos de Azure con el equilibrador de carga](../load-balancer/load-balancer-arm.md)
        
        {
          "apiVersion": "2015-06-15",
          "name": "[variables('loadBalancerName')]",
          "type": "Microsoft.Network/loadBalancers",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "[concat('Microsoft.Network/publicIPAddresses/', variables('publicIP1'))]"
          ],
          "properties": {
            "frontendIPConfigurations": [
              {
                "name": "loadBalancerFrontEnd",
                "properties": {
                  "publicIPAddress": {
                    "id": "[variables('publicIPAddressID1')]"
                  }
                }
              }
            ],
            "backendAddressPools": [
              {
                "name": "bepool1"
              }
            ],
            "inboundNatPools": [
              {
                "name": "natpool1",
                "properties": {
                  "frontendIPConfiguration": {
                    "id": "[variables('frontEndIPConfigID')]"
                  },
                  "protocol": "tcp",
                  "frontendPortRangeStart": 50000,
                  "frontendPortRangeEnd": 50500,
                  "backendPort": 22
                }
              }
            ]
          }
        },

8. Agregue el recurso de interfaz de red que utiliza la máquina virtual independiente. Dado que las máquinas de un conjunto de escalado de máquinas virtuales no son accesibles directamente mediante una dirección IP pública, se crea una máquina virtual independiente en la misma red virtual como conjunto de escalado y se usa para tener acceso remoto a las máquinas del conjunto.

        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/networkInterfaces",
          "name": "[variables('nicName1')]",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "[concat('Microsoft.Network/publicIPAddresses/', variables('publicIP2'))]",
            "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
          ],
          "properties": {
            "ipConfigurations": [
              {
                "name": "ipconfig1",
                "properties": {
                  "privateIPAllocationMethod": "Dynamic",
                  "publicIPAddress": {
                    "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIP2'))]"
                  },
                  "subnet": {
                    "id": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Network/virtualNetworks/',variables('virtualNetworkName'),'/subnets/',variables('subnetName'))]"
                  }
                }
              }
            ]
          }
        },

9. Agregue la máquina virtual en la misma red que el conjunto de escalado.

        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Compute/virtualMachines",
          "name": "[parameters('vmName')]",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "storageLoop",
            "[concat('Microsoft.Network/networkInterfaces/', variables('nicName1'))]"
          ],
          "properties": {
            "hardwareProfile": {
              "vmSize": "[variables('vmSize')]"
            },
            "osProfile": {
              "computername": "[parameters('vmName')]",
              "adminUsername": "[parameters('adminUsername')]",
              "adminPassword": "[parameters('adminPassword')]"
            },
            "storageProfile": {
              "imageReference": {
                "publisher": "[variables('imagePublisher')]",
                "offer": "[variables('imageOffer')]",
                "sku": "[variables('imageVersion')]",
                "version": "latest"
              },
              "osDisk": {
                "name": "osdisk1",
                "vhd": {
                  "uri":  "[concat('https://',parameters('resourcePrefix'),'sa.blob.core.windows.net/vhds/',parameters('resourcePrefix'),'osdisk1.vhd')]"
                },
                "caching": "ReadWrite",
                "createOption": "FromImage"
              }
            },
            "networkProfile": {
              "networkInterfaces": [
                {
                  "id": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName1'))]"
                }
              ]
            }
          }
        },

10.	Agregue el recurso de conjunto de escala de máquinas virtuales y especifique la extensión de diagnósticos que está instalada en todas las máquinas virtuales del conjunto de escala. Muchos de los valores para este recurso son similares al recurso de máquina virtual. Las principales diferencias son la adición del elemento de capacidad, que especifica cuántas máquinas virtuales se deben inicializar en el conjunto de escalado, y upgradePolicy, que especifica cómo se realizan las actualizaciones a las máquinas virtuales en el conjunto de escalado. El conjunto de escalado no se crea hasta que todas las cuentas de almacenamiento se crean como se especifica en el elemento dependsOn.

            {
              "type": "Microsoft.Compute/virtualMachineScaleSets",
              "apiVersion": "2016-03-30",
              "name": "[parameters('vmSSName')]",
              "location": "[resourceGroup().location]",
              "dependsOn": [
                "storageLoop",
                "[concat('Microsoft.Network/loadBalancers/', variables('loadBalancerName'))]",
                "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
              ],
              "sku": {
                "name": "[variables('vmSize')]",
                "tier": "Standard",
                "capacity": "[parameters('instanceCount')]"
              }
              "properties": {
                "upgradePolicy": {
                  "mode": "Manual"
                },
                "virtualMachineProfile": {
                  "storageProfile": {
                    "osDisk": {
                      "vhdContainers": [
                        "[concat('https://', parameters('resourcePrefix'), 'a.blob.core.windows.net/vmss')]",
                        "[concat('https://', parameters('resourcePrefix'), 'g.blob.core.windows.net/vmss')]",
                        "[concat('https://', parameters('resourcePrefix'), 'm.blob.core.windows.net/vmss')]",
                        "[concat('https://', parameters('resourcePrefix'), 's.blob.core.windows.net/vmss')]",
                        "[concat('https://', parameters('resourcePrefix'), 'y.blob.core.windows.net/vmss')]"
                      ],
                      "name": "vmssosdisk",
                      "caching": "ReadOnly",
                      "createOption": "FromImage"
                    },
                    "imageReference": {
                      "publisher": "[variables('imagePublisher')]",
                      "offer": "[variables('imageOffer')]",
                      "sku": "[variables('imageVersion')]",
                      "version": "latest"
                    }
                  },
                  "osProfile": {
                    "computerNamePrefix": "[parameters('vmSSName')]",
                    "adminUsername": "[parameters('adminUsername')]",
                    "adminPassword": "[parameters('adminPassword')]"
                  },
                  "networkProfile": {
                    "networkInterfaceConfigurations": [
                      {
                        "name": "[variables('nicName2')]",
                        "properties": {
                          "primary": "true",
                          "ipConfigurations": [
                            {
                              "name": "ip1",
                              "properties": {
                                "subnet": {
                                  "id": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Network/virtualNetworks/',variables('virtualNetworkName'),'/subnets/',variables('subnetName'))]"
                                },
                                "loadBalancerBackendAddressPools": [
                                  {
                                    "id": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Network/loadBalancers/',variables('loadBalancerName'),'/backendAddressPools/bepool1')]"
                                  }
                                ],
                                "loadBalancerInboundNatPools": [
                                  {
                                    "id": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Network/loadBalancers/',variables('loadBalancerName'),'/inboundNatPools/natpool1')]"
                                  }
                                ]
                              }
                            }
                          ]
                        }
                      }
                    ]
                  },
                  "extensionProfile": {
                    "extensions": [
                      {
                        "name":"LinuxDiagnostic",
                        "properties": {
                          "publisher":"Microsoft.OSTCExtensions",
                          "type":"LinuxDiagnostic",
                          "typeHandlerVersion":"2.1",
                          "autoUpgradeMinorVersion":false,
                          "settings": {
                            "xmlCfg":"[base64(concat(variables('wadcfgxstart'),variables('wadmetricsresourceid'),variables('wadcfgxend')))]",
                            "storageAccount":"[variables('diagnosticsStorageAccountName')]"
                          },
                          "protectedSettings": {
                            "storageAccountName":"[variables('diagnosticsStorageAccountName')]",
                            "storageAccountKey":"[listkeys(variables('accountid'), '2015-06-15').key1]",
                            "storageAccountEndPoint":"https://core.windows.net"
                          }
                        }
                      }
                    ]
                  }
                }
              }
            },

11.	Agregue el recurso autoscaleSettings que define cómo el conjunto de escala se ajusta según el uso del procesador en las máquinas del conjunto.
        
            {
              "type": "Microsoft.Insights/autoscaleSettings",
              "apiVersion": "2015-04-01",
              "name": "[concat(parameters('resourcePrefix'),'as1')]",
              "location": "[resourceGroup().location]",
              "dependsOn": [
                "[concat('Microsoft.Compute/virtualMachineScaleSets/',parameters('vmSSName'))]"
              ],
              "properties": {
                "enabled": true,
                "name": "[concat(parameters('resourcePrefix'),'as1')]",
                "profiles": [
                  {
                    "name": "Profile1",
                    "capacity": {
                      "minimum": "1",
                      "maximum": "10",
                      "default": "1"
                    },
                    "rules": [
                      {
                        "metricTrigger": {
                          "metricName": "\\Processor\\PercentProcessorTime",
                          "metricNamespace": "",
                          "metricResourceUri": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Compute/virtualMachineScaleSets/',parameters('vmSSName'))]",
                          "timeGrain": "PT1M",
                          "statistic": "Average",
                          "timeWindow": "PT5M",
                          "timeAggregation": "Average",
                          "operator": "GreaterThan",
                          "threshold": 50.0
                        },
                        "scaleAction": {
                          "direction": "Increase",
                          "type": "ChangeCount",
                          "value": "1",
                          "cooldown": "PT5M"
                        }
                      }
                    ]
                  }
                ],
                "targetResourceUri": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/', resourceGroup().name,'/providers/Microsoft.Compute/virtualMachineScaleSets/',parameters('vmSSName'))]"
              }
            }
    
    Para este tutorial, éstos son los valores importantes:
    
    - **metricName**: este es el mismo que el contador de rendimiento que definimos en la variable wadperfcounter. Con esta variable, la extensión Diagnósticos recopila los datos del contador **Processor\\PercentProcessorTime**.
    - **metricResourceUri**: este es el identificador de recursos del conjunto de escala de máquinas virtuales.
    - **timeGrain**: ésta es la granularidad de las métricas que se recopilan. En esta plantilla, se establece en 1 minuto.
    - **statistic**: determina cómo se combinan las métricas para dar cabida a la acción de escalado automático. Los valores posibles son: Average, Min y Max. En esta plantilla estamos buscando el uso total de CPU promedio entre las máquinas virtuales del conjunto de escala.
    - **timeWindow**: este es el intervalo de tiempo en que se recopilan los datos de instancia. Debe estar comprendido entre 5 minutos y 12 horas.
    - **timeAggregation**: este determina la manera en que se deberían combinar los datos recopilados en el tiempo. El valor predeterminado es Average. Los valores posibles son: Average, Minimum, Maximum, Last, Total, Count.
    - **operator**: este es el operador que se utiliza para comparar los datos de métrica y el umbral. Los valores posibles son: Equals, NotEquals, GreaterThan, GreaterThanOrEqual, LessThan, LessThanOrEqual.
    - **threshold**: este es el valor que desencadena la acción de escalado. En esta plantilla, los equipos se agregan al conjunto de escala que se establece cuando el uso promedio de CPU entre máquinas del conjunto es más de un 50%.
    - **direction**: este determina la acción que se realiza cuando se alcanza el valor de umbral. Los valores posibles son Increase o Decrease. En esta plantilla, se aumenta el número de máquinas virtuales en el conjunto de escala si el umbral es más de un 50% en la ventana de tiempo definido.
    - **type**: este es el tipo de acción que debe realizarse y que se debe establecer en ChangeCount.
    - **value**: este es el número de máquinas virtuales que se agregan o se quitan del conjunto de escala. Este valor debe ser 1 o un valor superior. El valor predeterminado es 1. En esta plantilla, el número de máquinas en el conjunto de escala aumenta en 1 cuando se alcanza el umbral.
    - **cooldown**: es la cantidad de tiempo de espera desde la última acción de escalada hasta antes de que se produzca la siguiente acción. Este valor debe estar comprendido entre 1 minuto y 1 semana.

12.	Guarde el archivo de plantilla.

## Paso 3: Carga de la plantilla en el almacenamiento

La plantilla se puede cargar desde la interfaz de línea de comandos, siempre y cuando conozca el nombre de cuenta y la clave principal de la cuenta de almacenamiento que creó en el paso 1.

1. En la interfaz de línea de comandos (Bash, Terminal, símbolo del sistema), ejecute estos comandos para establecer las variables de entorno que se necesitan para acceder a la cuenta de almacenamiento:

		export AZURE_STORAGE_ACCOUNT={account_name}
		export AZURE_STORAGE_ACCESS_KEY={key}

	Puede obtener la clave haciendo clic en el icono de llave al ver el recurso de la cuenta de almacenamiento en el portal de Azure. Cuando use un símbolo del sistema de Windows, escriba **set** en lugar de export.

2. Cree el contenedor para almacenar la plantilla:

		azure storage container create -p Blob templates

3. Cargue el archivo de plantillas en el nuevo contenedor.

		azure storage blob upload VMSSTemplate.json templates VMSSTemplate.json

## Paso 4: Implementación de la plantilla

Ahora que ha creado la plantilla, puede empezar a implementar los recursos. Use este comando para iniciar el proceso:

	azure group deployment create --template-uri https://vmsstestsa.blob.core.windows.net/templates/VMSSTemplate.json vmsstestrg1 vmsstestdp1

Cuando presione Entrar, se le pedirá que proporcione valores para las variables que asignó. Proporcione estos valores:

	vmName: vmsstestvm1
	vmSSName: vmsstest1
	instanceCount: 5
	adminUserName: vmadmin1
	adminPassword: VMpass1
	resourcePrefix: vmsstest

Para que todos los recursos se implementen correctamente se tardará unos 15 minutos.

>[AZURE.NOTE]También puede hacer uso de la capacidad del portal para implementar los recursos. Para ello, use este vínculo: https://portal.azure.com/#create/Microsoft.Template/uri/<link to VM Scale Set JSON template>

## Paso 5: Supervisión de recursos

Puede obtener información acerca de los conjuntos de escala de máquinas virtuales mediante estos métodos:

 - El portal de Azure: actualmente puede obtener una cantidad limitada de información mediante el portal.
 - El [explorador de recursos de Azure](https://resources.azure.com/): esta es la mejor herramienta para explorar el estado actual del conjunto de escala. Siga esta ruta de acceso y debería ver la vista de instancia del conjunto de escala que creó:

		subscriptions > {your subscription} > resourceGroups > vmsstestrg1 > providers > Microsoft.Compute > virtualMachineScaleSets > vmsstest1 > virtualMachines

 - CLI de Azure. Use este comando para obtener información:

		azure resource show -n vmsstest1 -r Microsoft.Compute/virtualMachineScaleSets -o 2015-06-15 -g vmsstestrg1

 - Conéctese a la máquina virtual de jumpbox igual que lo haría con cualquier otra máquina y, a continuación, podrá obtener acceso remoto a las máquinas virtuales del conjunto de escala para supervisar los procesos individuales.

>[AZURE.NOTE]Puede encontrar una API de REST completa para obtener más información sobre los conjuntos de escala en [Conjuntos de escala de máquinas virtuales](https://msdn.microsoft.com/library/mt589023.aspx).

## Paso 6: Eliminación de recursos

Dado que se le cobrará por los recursos utilizados en Azure, siempre es conveniente eliminar los recursos que ya no sean necesarios. No es necesario eliminar por separado cada recurso de un grupo de recursos. Puede eliminar el grupo de recursos y se eliminarán automáticamente todos sus recursos.

		azure group delete vmsstestrg1

## Pasos siguientes

- Vea ejemplos de características de supervisión de Azure Insights en [Ejemplos de inicio rápido de CLI multiplataforma de Azure Insights](../azure-portal/insights-cli-samples.md).
- Obtenga información sobre las características de notificación en [Uso de acciones de escalado automático para enviar notificaciones de alerta por correo electrónico y Webhook en Azure Insights](../azure-portal/insights-autoscale-to-webhook-email.md) y [Uso de registros de auditoría para enviar notificaciones de alerta por correo electrónico y webhook en Azure Insights](../azure-portal/insights-auditlog-to-webhook-email.md).
- Consulte la plantilla [Autoscale a VM Scale Set running a Ubuntu/Apache/PHP app (Escalado automático de un conjunto de escala de VM que ejecuta una aplicación Ubuntu, Apache o PHP)](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-lapstack-autoscale), en la que se define una pila LAMP para ejercer la funcionalidad de escalado automático de los conjuntos de escalas de máquina virtual.

<!---HONumber=AcomDC_0810_2016-->