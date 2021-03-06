<properties
   pageTitle="Crear y cargar una imagen de máquina virtual de FreeBSD | Microsoft Azure"
   description="Aprenda a crear y cargar un disco duro virtual (VHD) que contenga el sistema operativo FreeBSD para crear una máquina virtual de Azure."
   services="virtual-machines-linux"
   documentationCenter=""
   authors="KylieLiang"
   manager="timlt"
   editor=""
   tags="azure-service-management"/>

<tags
   ms.service="virtual-machines-linux"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-linux"
   ms.workload="infrastructure-services"
   ms.date="08/29/2016"
   ms.author="kyliel"/>

# Creación y carga de un VHD de FreeBSD en Azure

En este artículo se muestra cómo crear y cargar un disco duro virtual (VHD) que contenga el sistema operativo FreeBSD. Después de cargarlo, puede utilizarlo como su propia imagen para crear una máquina virtual (VM) en Azure.

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)]


## Requisitos previos
En este artículo se supone que tiene los siguientes elementos:

- **Una suscripción de Azure**: si no tiene una cuenta, puede crear una en un par de minutos. Si tiene una suscripción a MSDN, consulte [Crédito mensual de Azure para suscriptores de Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/). De lo contrario, obtenga información sobre cómo [crear una cuenta de prueba gratuita](https://azure.microsoft.com/pricing/free-trial/).

- **Herramientas de Azure PowerShell**: el módulo Azure PowerShell debe estar instalado y configurado para utilizar su suscripción de Azure. Para descargar el módulo, consulte la página de [descargas de Azure](https://azure.microsoft.com/downloads/). Hay disponible aquí un tutorial que describe cómo instalar y configurar el módulo. Use el cmdlet de [descargas de Azure](https://azure.microsoft.com/downloads/) para cargar el VHD.

- **Sistema operativo FreeBSD instalado en un archivo .vhd**: se debe instalar un sistema operativo FreeBSD compatible en un disco duro virtual. Existen varias herramientas para crear archivos .vhd. Por ejemplo, puede utilizar una solución de virtualización como Hyper-V para crear el archivo .vhd e instalar el sistema operativo. Para obtener instrucciones sobre cómo instalar y utilizar Hyper-V, consulte [Instalar Hyper-V y crear una máquina virtual](http://technet.microsoft.com/library/hh846766.aspx).

> [AZURE.NOTE] el reciente formato VHDX no se admite en Azure. Puede convertir el disco al formato VHD mediante el Administrador de Hyper-V o el cmdlet [convert-vhd](https://technet.microsoft.com/library/hh848454.aspx). Además, hay un [tutorial en MSDN sobre cómo utilizar FreeBSD con Hyper-V](http://blogs.msdn.com/b/kylie/archive/2014/12/25/running-freebsd-on-hyper-v.aspx).

Esta tarea incluye los cinco pasos siguientes.

## Paso 1: Preparación de la imagen que se va a cargar

En la máquina virtual en la que se instaló el sistema operativo FreeBSD, realice los procedimientos siguientes:

1. Habilite DHCP.

		# echo 'ifconfig_hn0="SYNCDHCP"' >> /etc/rc.conf
		# service netif restart

2. Habilite SSH.

    SSH se activa de forma predeterminada después de la instalación desde el disco. Si no está habilitado por algún motivo, o si utiliza VHD de FreeBSD directamente, escriba lo siguiente:

		# echo 'sshd_enable="YES"' >> /etc/rc.conf
		# ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
		# ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
		# service sshd restart

3. Configure la consola de serie.

		# echo 'console="comconsole vidconsole"' >> /boot/loader.conf
		# echo 'comconsole_speed="115200"' >> /boot/loader.conf

4. Instale sudo.

    La cuenta raíz está deshabilitada en Azure. Esto significa que deberá usar sudo con un usuario sin privilegios para ejecutar comandos con privilegios elevados.

		# pkg install sudo
;
5. Requisitos previos del agente de Azure.

		# pkg install python27  
		# pkg install Py27-setuptools27   
		# ln -s /usr/local/bin/python2.7 /usr/bin/python   
		# pkg install git

6. Instale el agente de Azure.

    Siempre puede encontrar la versión más reciente del agente de Azure en [github](https://github.com/Azure/WALinuxAgent/releases). La versión 2.0.10 + admite oficialmente FreeBSD 10 y 10.1 y la versión 2.1.4 admite oficialmente FreeBSD 10.2 y versiones posteriores.

		# git clone https://github.com/Azure/WALinuxAgent.git  
		# cd WALinuxAgent  
		# git tag  
		…
		WALinuxAgent-2.0.16
		…
		v2.1.4
		v2.1.4.rc0
		v2.1.4.rc1

    Para 2.0, usaremos la 2.0.16 como ejemplo:

		# git checkout WALinuxAgent-2.0.16
		# python setup.py install  
		# ln -sf /usr/local/sbin/waagent /usr/sbin/waagent  

    Para 2.1, utilizaremos la 2.1.4 como ejemplo:

		# git checkout v2.1.4
		# python setup.py install  
		# ln -sf /usr/local/sbin/waagent /usr/sbin/waagent  
		# ln -sf /usr/local/sbin/waagent2.0 /usr/sbin/waagent2.0

    >[AZURE.IMPORTANT] Después de instalar el agente de Azure, es recomendable comprobar que está ejecutándose:

		# waagent -version
		WALinuxAgent-2.1.4 running on freebsd 10.3
		Python: 2.7.11
		# service –e | grep waagent
		/etc/rc.d/waagent
		# cat /var/log/waagent.log

7. Desaprovisione el sistema.

    Desaprovisione el sistema para limpiarlo y dejarlo adecuado para un reaprovisionamiento. El comando siguiente también elimina la última cuenta de usuario aprovisionada y los datos asociados:

		# echo "y" |  /usr/local/sbin/waagent -deprovision+user  
		# echo  'waagent_enable="YES"' >> /etc/rc.conf

    Ahora ya puede apagar la máquina virtual.

## Paso 2: Creación de una cuenta de almacenamiento en Azure ##

Necesita una cuenta de almacenamiento de Azure para cargar un archivo .vhd, que se puede usar para crear una máquina virtual. Puede utilizar el portal clásico de Azure para crear una cuenta de almacenamiento.

1. Inicie sesión en el [Portal de Azure clásico](https://manage.windowsazure.com).

2. En la barra de comandos, haga clic en **Nuevo**.

3. Seleccione **Servicios de datos** > **Almacenamiento** > **Creación rápida**.

	![Crear rápidamente una cuenta de almacenamiento](./media/virtual-machines-linux-classic-freebsd-create-upload-vhd/Storage-quick-create.png)

4. Rellene los campos de la manera siguiente:

	- En el campo **URL**, escriba un nombre de subdominio que vaya a usar en la URL de la cuenta de almacenamiento. Esta entrada puede contener de 3 a 24 números y letras minúsculas. Este nombre se convierte en el nombre del host dentro de la dirección URL que se usó para direccionar el Almacenamiento de blobs de Azure, el Almacenamiento en cola de Azure o los recursos de Almacenamiento de tablas de Azure de la suscripción.

	- En la lista desplegable **Ubicación/grupo de afinidad**, seleccione una **ubicación o grupo de afinidad** para la cuenta de almacenamiento. Un grupo de afinidad le permite colocar sus servicios y almacenamiento en la nube en el mismo centro de datos.

	- En el campo **Replicación**, decida si va a utilizar la replicación **Redundancia geográfica** para la cuenta de almacenamiento. La replicación geográfica está activada de forma predeterminada. Esta opción replica los datos en una ubicación secundaria, sin coste, por lo que su almacenamiento conmuta por error a esa ubicación si se produce un error importante en la ubicación principal. La ubicación secundaria se asigna automáticamente y no se puede cambiar. Si necesita más control sobre la ubicación del almacenamiento en la nube debido a requisitos legales o las directivas de su organización, puede desactivar la replicación geográfica. Sin embargo, tenga en cuenta que si más tarde activa la replicación geográfica, deberá pagar una cuota de transferencia de datos puntual para replicar los datos existentes en la ubicación secundaria. Los servicios de almacenamiento sin replicación geográfica se ofrecen con descuento. Puede encontrar más detalles sobre la replicación geográfica de las cuentas de almacenamiento en: [Creación, administración o eliminación de una cuenta de almacenamiento](../storage-create-storage-account/#replication-options).

	![Escribir los detalles de la cuenta de almacenamiento](./media/virtual-machines-linux-classic-freebsd-create-upload-vhd/Storage-create-account.png)


5. Haga clic en **Crear cuenta de almacenamiento**. La cuenta aparece ahora en **Almacenamiento**.

	![Cuenta de almacenamiento creada correctamente](./media/virtual-machines-linux-classic-freebsd-create-upload-vhd/Storagenewaccount.png)

6. Después, cree un contenedor para los archivos .vhd que ha cargado. Seleccione el nombre de la cuenta de almacenamiento y, a continuación, elija **Contenedores**.

	![Detalles de la cuenta de almacenamiento](./media/virtual-machines-linux-classic-freebsd-create-upload-vhd/storageaccount_detail.png)

7. Seleccione **Crear un contenedor**.

	![Detalles de la cuenta de almacenamiento](./media/virtual-machines-linux-classic-freebsd-create-upload-vhd/storageaccount_container.png)

8. En el campo **Nombre**, escriba un nombre para el contenedor. A continuación, en el menú desplegable **Acceso**, seleccione el tipo de directiva de acceso que desee.

	![Nombre del contenedor](./media/virtual-machines-linux-classic-freebsd-create-upload-vhd/storageaccount_containervalues.png)

    > [AZURE.NOTE] De manera predeterminada, el contenedor es privado y solo puede acceder a él el propietario de la cuenta. Para permitir el acceso de lectura público a los blobs del contenedor, pero no a las propiedades y metadatos del contenedor, use la opción **Blob público**. Para permitir el acceso de lectura público completo para el contenedor y los blobs, utilice la opción **Contenedor público**.

## Paso 3: Preparación de la conexión a Azure

Antes de cargar el archivo .vhd, debe establecer una conexión segura entre el equipo y la suscripción de Azure. Para ello, puede utilizar el método de Azure Active Directory (Azure AD) o el método del certificado.

### Uso del método de Azure AD para cargar un archivo .vhd

1. Abra la consola de Azure PowerShell.

2. Escriba el siguiente comando: `Add-AzureAccount`

	Este comando abre una ventana de inicio de sesión donde podrá iniciar sesión con su cuenta profesional o educativa.

	![Ventana de PowerShell](./media/virtual-machines-linux-classic-freebsd-create-upload-vhd/add_azureaccount.png)

3. Azure autentica y guarda las credenciales. A continuación, cierra la ventana.

### Uso del método del certificado para cargar un archivo .vhd

1. Abra la consola de Azure PowerShell.

2. Escriba: `Get-AzurePublishSettingsFile`.

3. Se abre una ventana del explorador que le solicita que descargue un archivo .publishsettings. Este archivo contiene información y un certificado para su suscripción de Azure.

	![Página de descarga del explorador](./media/virtual-machines-linux-classic-freebsd-create-upload-vhd/Browser_download_GetPublishSettingsFile.png)

3. Guarde el archivo .publishsettings.

4. Escriba: `Import-AzurePublishSettingsFile <PathToFile>`, donde `<PathToFile>` es la ruta completa al archivo .publishsettings.

   Para obtener información, consulte [Get started with Azure cmdlets](http://msdn.microsoft.com/library/windowsazure/jj554332.aspx) (Introducción a los cmdlets de Azure).

   Para más información sobre cómo instalar Azure PowerShell, consulte [Cómo instalar y configurar Azure PowerShell](../powershell-install-configure.md).

## Paso 4: Carga del archivo .vhd

Cuando carga el archivo .vhd, puede colocarlo en cualquier parte del Almacenamiento de blobs. A continuación se muestran algunos términos que se van a utilizar al cargar el archivo:
-  **URLAlmacenamientoDeBlobs** es la dirección URL de la cuenta de almacenamiento que ha creado en el paso 2.
-  **SuCarpetaDeImágenes** es el contenedor de Almacenamiento de blobs en el que desea almacenar las imágenes.
- **VHDName** es la etiqueta que aparece en el Portal de Azure clásico para identificar el disco duro virtual.
- **PathToVHDFile** es la ruta de acceso completa y el nombre del archivo .vhd.


Desde la ventana de Azure PowerShell que ha usado en el paso anterior, escriba:

		Add-AzureVhd -Destination "<BlobStorageURL>/<YourImagesFolder>/<VHDName>.vhd" -LocalFilePath <PathToVHDFile>

## Paso 5: Creación de una máquina virtual con el archivo .vhd cargado
Después de cargar el archivo .vhd, puede agregarlo como imagen a la lista de imágenes personalizadas que están asociadas con su suscripción y crear una máquina virtual con esta imagen personalizada.

1. Desde la ventana de Azure PowerShell que ha usado en el paso anterior, escriba:

		Add-AzureVMImage -ImageName <Your Image's Name> -MediaLocation <location of the VHD> -OS <Type of the OS on the VHD>

    > [AZURE.NOTE]Utilice Linux como tipo de sistema operativo. La versión actual de Azure PowerShell acepta solo "Linux" o "Windows" como parámetro.

2. Tras completar los pasos anteriores, la nueva imagen aparecerá en la lista cuando elija la pestaña **Imágenes** en el Portal de Azure clásico.

    ![Elija una imagen](./media/virtual-machines-linux-classic-freebsd-create-upload-vhd/addfreebsdimage.png)

3. Cree una máquina virtual desde la galería. Esta nueva imagen ahora está disponible en **Mis imágenes**.
4. Seleccione la nueva imagen. Siga las indicaciones para configurar un nombre de host, una contraseña, una clave SSH, etc.

	![Imagen personalizada](./media/virtual-machines-linux-classic-freebsd-create-upload-vhd/createfreebsdimageinazure.png)

4. Una vez completado el aprovisionamiento, verá que la máquina virtual de FreeBSD se ejecuta en Azure.

	![Imagen de FreeBSD en Azure](./media/virtual-machines-linux-classic-freebsd-create-upload-vhd/freebsdimageinazure.png)

<!---HONumber=AcomDC_0831_2016-->