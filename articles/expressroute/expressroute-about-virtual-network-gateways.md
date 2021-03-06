<properties 
   pageTitle="Acerca de las puertas de enlace de red virtual de ExpressRoute | Microsoft Azure"
   description="Conozca sobre las puertas de enlace de red virtual para ExpressRoute."
   services="expressroute"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager, azure-service-management"/>
<tags 
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/21/2016"
   ms.author="cherylmc" />

# Acerca de las puertas de enlace de red virtual para ExpressRoute


Una puerta de enlace de red virtual se usa para enviar tráfico de red entre redes virtuales y ubicaciones locales de Azure. Cuando configure una conexión ExpressRoute, debe crear y configurar una puerta de enlace de red virtual y una conexión de puerta de enlace de red virtual.

Al crear una puerta de enlace de red virtual, se especifican varios valores de configuración. Uno de los valores de configuración necesarios especifica si la puerta de enlace se usará para el tráfico de ExpressRoute o de VPN Gateway. En el modelo de implementación de Resource Manager, el valor es '-GatewayType'.

Cuando se envíe el tráfico de red en una conexión privada dedicada, use el tipo de puerta de enlace "ExpressRoute". Esto también se conoce como puerta de enlace de ExpressRoute. Cuando se envíe el tráfico de red cifrado a través de una conexión pública, use el tipo de puerta de enlace "Vpn". Esto se conoce como puerta de enlace de VPN. Las conexiones de sitio a sitio, de punto a sitio y de red virtual a red virtual utilizan una puerta de enlace de VPN.

Cada red virtual tiene una única puerta de enlace de red virtual por cada tipo de puerta de enlace. Por ejemplo, puede tener una puerta de enlace de una red virtual que use -GatewayType Vpn y otra que use -GatewayType ExpressRoute. Este artículo se centra en la puerta de enlace de red virtual de ExpressRoute.

## <a name="gwsku"></a>SKU de puerta de enlace

[AZURE.INCLUDE [expressroute-gwsku-include](../../includes/expressroute-gwsku-include.md)]

Si usa Azure Portal para crear una puerta de enlace de red virtual de Resource Manager, dicha puerta de enlace se configura de forma predeterminada mediante SKU estándar. Actualmente, no se pueden especificar otras SKU para el modelo de implementación de Resource Manager en Azure Portal. Sin embargo, después de crear la puerta de enlace, puede actualizar a una SKU de puerta de enlace más eficaz (por ejemplo, de básica o estándar a alto rendimiento) mediante el cmdlet 'Resize-AzureRmVirtualNetworkGateway' de PowerShell.

###  <a name="aggthroughput"></a>Rendimiento agregado estimado por SKU de puerta de enlace


En la tabla siguiente se muestran los tipos de puerta de enlace y el rendimiento agregado estimado. Esta tabla se aplica a los modelos de implementación del Administrador de recursos y clásico.

[AZURE.INCLUDE [expressroute-table-aggthroughput](../../includes/expressroute-table-aggtput-include.md)]


## <a name="resources"></a>API de REST y cmdlets de PowerShell

Para más información sobre recursos técnicos y requisitos de sintaxis específicos al usar API de REST y cmdlets de PowerShell para configuraciones de puerta de enlace de red virtual, consulte las páginas siguientes:

|**Clásico** | **Resource Manager**|
|-----|----|
|[PowerShell](https://msdn.microsoft.com/library/mt270335.aspx)|[PowerShell](https://msdn.microsoft.com/library/mt163510.aspx)|
|[API DE REST](https://msdn.microsoft.com/library/jj154113.aspx)|[API DE REST](https://msdn.microsoft.com/library/mt163859.aspx)|


## Pasos siguientes

Consulte [Información técnica de ExpressRoute](expressroute-introduction.md) para más información sobre configuraciones de conexión disponibles.







 

<!---HONumber=AcomDC_0921_2016-->