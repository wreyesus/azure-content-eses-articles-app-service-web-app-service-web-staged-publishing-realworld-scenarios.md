<properties
	pageTitle="Replicación de máquinas virtuales locales de VMware o de servidores físicos en un sitio secundario | Microsoft Azure"
	description="Use este artículo para replicar máquinas virtuales de VMware o servidores físicos de Windows o Linux en un sitio secundario con Azure Site Recovery."
	services="site-recovery"
	documentationCenter=""
	authors="nsoneji"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.workload="backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="nisoneji"/>


# Replicación de máquinas virtuales locales de VMware o de servidores físicos en un sitio secundario


## Información general

InMage Scout en Azure Site Recovery proporciona características de replicación en tiempo real entre los sitios locales de VMware. InMage Scout se incluye en las suscripciones al servicio Azure Site Recovery.


## Requisitos previos

**Cuenta de Azure**: necesitará una cuenta de [Microsoft Azure](https://azure.microsoft.com/). Puede comenzar con una [evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/). [Más información](https://azure.microsoft.com/pricing/details/site-recovery/) sobre los precios de Site Recovery.


## Paso 1: Creación de un almacén

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com).
2. Haga clic en **Servicios de datos** > **Servicios de recuperación** y **Almacén de Site Recovery**.
3. Haga clic en **Crear nuevo** > **Creación rápida**.
4. En **Nombre**, escriba un nombre descriptivo para identificar el almacén.
5. En **Región**, seleccione la región geográfica del almacén. Para ver las regiones admitidas, consulte [Precios de Azure Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/).

Compruebe la barra de estado para confirmar que el almacén se ha creado correctamente. El almacén aparecerá como **Activo** en la página principal de **Servicios de recuperación**.

## Paso 2: Configuración del almacén y descarga de componentes InMage Scout

1. Haga clic en **Crear almacén**.
2. En la página **Servicios de recuperación**, haga clic en el almacén para abrir la página **Inicio rápido**.
3. En la lista desplegable, seleccione **Entre dos sitios locales de VMware**.
4. Descargue a InMage Scout. Los archivos de instalación para todos los componentes necesarios están en el archivo .zip descargado.


## Paso 3: Instalación de actualizaciones de componentes

Obtenga información acerca de las [actualizaciones](#updates) más recientes. Deberá instalar los archivos de actualización en los servidores en el orden siguiente:

1. Servidor RX si hay alguno
2. Servidores de configuración
3. Servidores de proceso
3. Servidores de destino maestros
4. Servidores de vContinuum
5. Servidor de origen (solo para Windows Server)

Instale las actualizaciones de la siguiente manera:

1. Descargue el archivo .zip de la [actualización](https://aka.ms/asr-scout-update3). Este archivo .zip contiene los archivos siguientes:

	-  RX_8.0.3.0_GA\_Update_3_6684045\_17Mar16.tar.gz
	-  CX\_Windows_8.0.3.0_GA\_Update_3_5048668\_16Mar16.exe
	-  UA\_Windows_8.0.3.0_GA\_Update_3_7101745\_04Apr16.exe
	-  UA\_RHEL6-64_8.0.3.0_GA\_Update_3_7101745\_04Apr16.zip
	-  vCon\_Windows_8.0.3.0_GA\_Update_3_6873369\_16Mar16.exe

2. Extraiga los archivos .zip.
3. **Para el servidor RX**: copie **RX_8.0.3.0_GA\_Update_3_6684045\_17Mar16.tar.gz** en el servidor RX y extráigalo. En la carpeta extraída, ejecute **/Install**.
4. **Para el servidor de configuración/servidor de proceso**: copie **CX\_Windows_8.0.3.0_GA\_Update_3_5048668\_16Mar16.exe** en el servidor de configuración y el servidor de proceso. Haga doble clic para ejecutarlo.
5. **Para el servidor de destino maestro de Windows**: para actualizar el agente unificado, copie **UA\_Windows_8.0.3.0_GA\_Update_3_7101745\_04Apr16.exe** en el servidor de destino maestro. Haga doble clic en él para ejecutarlo. Tenga en cuenta que el agente unificado también es aplicable en el servidor de origen. Debe instalarlo también en el servidor de origen, como se menciona más adelante en esta lista.
6. **Para el servidor de destino maestro de Linux**: para actualizar el agente unificado, copie **UA\_RHEL6-64_8.0.3.0_GA\_Update_3_7101745\_04Apr16.zip** en el servidor de destino maestro y extráigalo. En la carpeta extraída, ejecute **/Install**.
7. **Para el servidor de vContinuum**: copie **vCon\_Windows_8.0.3.0_GA\_Update_3_6873369\_16Mar16.exe** en el servidor de vContinuum. Asegúrese de que ha cerrado el asistente de vContinuum. Haga doble clic en el archivo para ejecutarlo.
8. **Para el servidor de origen de Windows**: para actualizar el agente unificado, copie **UA\_Windows_8.0.3.0_GA\_Update_3_7101745\_04Apr16.exe** en el servidor de origen. Haga doble clic en él para ejecutarlo.

## Paso 4: Configuración de la replicación
1. Configure la replicación entre los sitios de origen y de destino de VMware.
2. Para obtener instrucciones, use la documentación de InMage Scout que se descarga con el producto. También se puede tener acceso a la documentación del modo indicado a continuación:

	- [Notas de la versión](https://aka.ms/asr-scout-release-notes)
	- [Matriz de compatibilidad](https://aka.ms/asr-scout-cm)
	- [Guía de usuario](https://aka.ms/asr-scout-user-guide)
	- [Guía de usuario de RX](https://aka.ms/asr-scout-rx-user-guide)
	- [Guía de instalación rápida](https://aka.ms/asr-scout-quick-install-guide)


## Actualizaciones

### Azure Site Recovery Scout 8.0.1 actualización 3
La actualización 3 incluye las siguientes revisiones de errores y mejoras:

- El servidor de configuración y RX no se pueden registrar en el almacén de Site Recovery cuando están detrás del proxy.
- El número de horas que no se cumple el objetivo de punto de recuperación (RPO) no se actualiza en el informe de mantenimiento.
- El servidor de configuración no se sincroniza con RX cuando los detalles del hardware ESX o de la red contienen cualquier carácter UTF-8.
- Los controladores de dominio de Windows Server 2008 R2 no arrancan después de la recuperación.
- La sincronización sin conexión no funciona según lo esperado.
- Después de la conmutación por error de la máquina virtual (VM), la eliminación de pares de replicación se detiene en la interfaz de usuario de CX durante mucho tiempo, y los usuarios no pueden completar la conmutación por recuperación ni reanudar la operación.
- Las operaciones de instantáneas globales realizadas por el trabajo de coherencia se han optimizado para ayudar a reducir las desconexiones de la aplicación como clientes de SQL.
- Se ha mejorado el rendimiento de la herramienta de coherencia (VACP.exe) al reducir el uso de memoria que se requiere para crear instantáneas en Windows.
- El servicio de instalación de inserción se bloquea cuando la contraseña tiene más de 16 caracteres.
- vContinuum no está comprobando y solicitando las nuevas credenciales de vCenter cuando se modifican.
- En Linux, al administrador de caché (cachemgr) de destino maestro no descarga los archivos del servidor de proceso, lo que genera una limitación de pares de replicación.
- Cuando el orden del disco físico del clúster de conmutación por error (MSCS) no es el mismo en todos los nodos, la replicación no se establece para algunos de los volúmenes del clúster. <br/>Tenga en cuenta que el clúster debe protegerse de nuevo para aprovechar esta revisión.
- La funcionalidad de SMTP no funciona como se esperaba después de actualizar RX de Scout 7.1 a Scout 8.0.1.
- Se han agregado más estadísticas en el registro de la operación de reversión para medir el tiempo que ha tardado en completarse.
- Se ha agregado compatibilidad para los sistemas operativos Linux en el servidor de origen:
	- Red Hat Enterprise Linux (RHEL) 6 actualización 7
	- CentOS 6 actualización 7
- Las interfaces de usuario de CX y RX ahora pueden mostrar la notificación para el par que entra en modo de mapa de bits.
- Las siguientes revisiones de seguridad se han agregado a RX:

**Descripción del problema**|**Procedimientos de implementación**
---|---
Omisión de la autorización mediante la alteración de parámetros|Se ha restringido el acceso a usuarios no aplicables.
Falsificación de solicitudes entre sitios|Se ha implementado el concepto de token de página que se genera aleatoriamente para cada página. <br/>De este modo, verá que: <li> solo hay una instancia de inicio de sesión único para el mismo usuario.</li><li>La actualización de la página no funciona, se redirigirá al panel.</li>
Carga de archivos malintencionados|Archivos restringidos a determinadas extensiones. Se permiten las siguientes extensiones: 7z, aiff, asf, avi, bmp, csv, doc, docx, fla, flv, gif, gz, gzip, jpeg, jpg, log, mid, mov, mp3, mp4, mpc, mpeg, mpg, ods, odt, pdf, png, ppt, pptx, pxd, qt, ram, rar, rm, rmi, rmvb, rtf, sdc, sitd, swf, sxc, sxw, tar, tgz, tif, tiff, txt, vsd, wav, wma, wmv, xls, xlsx, xml y zip.
Scripting entre sitios persistente | Se han agregado validaciones de entrada.


>[AZURE.NOTE]
>
>-	Todas las actualizaciones de Site Recovery son acumulativas. La actualización 3 tiene todas las revisiones de las actualizaciones 1 y 2. La actualización 3 se puede aplicar directamente en 8.0.1 GA.
>-	Las actualizaciones del servidor de configuración y RX no se pueden revertir tras aplicarse en el sistema.

### Azure Site Recovery Scout 8.0.1 actualización 2 (actualización del 3 de diciembre de 2015)

Las revisiones de la actualización 2 incluyen:

- **Servidor de configuración**: corrige un problema que impedía que la característica de disponibilidad gratuita durante 31 días funcionase según lo esperado al registrar el servidor de configuración en Site Recovery.
- **Agente unificado**: corrige un problema en la actualización 1, cuyo resultado era que la actualización no se instalaba en el servidor de destino maestro al actualizar de la versión 8.0 a 8.0.1.


### Azure Site Recovery Scout 8.0.1 actualización 1

La actualización 1 incluye las siguientes revisiones de errores y características nuevas:

- 31 días de protección gratuita por instancia de servidor. Esto le permite probar la funcionalidad o configurar una prueba de concepto.
	- Todas las operaciones en el servidor, incluida la conmutación por error y la conmutación por recuperación, son gratuitas durante los primeros 31 días a partir del momento en que un servidor primero es protegido por primera vez mediante Site Recovery Scout.
	- Desde el 32º día en adelante, a cada servidor protegido se le cargará la tarifa de instancia estándar de la protección de Azure Site Recovery en un sitio propiedad de un cliente.
	- En cualquier momento, el número de servidores protegidos que se están cargando actualmente está disponible en la página de Panel del almacén de Azure Site Recovery.
- Compatibilidad agregada para la interfaz de la línea de comandos (vCLI) de vSphere 5.5 Update 2.
- Compatibilidad agregada para los sistemas operativos Linux en el servidor de origen:
	- RHEL 6 Update 6
	- RHEL 5 Update 11
	- CentOS 6 Update 6
	- CentOS 5 Update 11
- Correcciones de errores para resolver los problemas siguientes:
	- Se produce un error en el registro del almacén para el servidor de configuración o el servidor RX.
	- Los volúmenes de clúster no aparecen del modo esperado cuando las máquinas virtuales agrupadas en clústeres se vuelven a proteger al reanudarse.
	- La conmutación por recuperación falla cuando el servidor de destino maestro se hospeda en un servidor ESXi diferente de las máquinas virtuales de producción locales.
	- Los permisos de archivo de configuración se cambian al actualizar a la versión 8.0.1, lo cual afecta a la protección y las operaciones.
	- El umbral de resincronización no se aplica del modo esperado, dando lugar a un comportamiento de replicación incoherente.
	- La configuración de RPO no aparece correctamente en la interfaz del servidor de configuración. El valor de datos sin comprimir muestra incorrectamente el valor comprimido.
	-  La operación de eliminación no efectúa la eliminación como se esperaba en el asistente de vContinuum y la replicación no se elimina de la interfaz del servidor de configuración.
	-  En el asistente de vContinuum, el disco deja automáticamente de estar seleccionado al hacer clic en **Detalles** en la vista de disco durante la protección de las máquinas virtuales MSCS.
	- Durante el escenario físico a virtual (P2V), algunos servicios HP requeridos, como CIMnotify y CqMgHost, no se mueven a manual en la recuperación de la máquina virtual. Esto provoca que se produzca un tiempo de arranque adicional.
	- Se produce un error en la protección de la máquina virtual Linux cuando hay más de 26 discos en el servidor de destino maestro.

## Pasos siguientes

Publique cualquier pregunta en el [Foro de servicios de recuperación de Azure](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr).

<!---HONumber=AcomDC_0921_2016-->