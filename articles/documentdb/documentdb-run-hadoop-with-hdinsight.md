<properties
	pageTitle="Ejecución de un trabajo de Hadoop con DocumentDB y HDInsight | Microsoft Azure"
	description="Obtenga información acerca de cómo ejecutar un trabajo de Hive, Pig y MapReduce simple con DocumentDB y Azure HDInsight."
	services="documentdb"
	authors="AndrewHoh"
	manager="jhubbard"
	editor="mimig"
	documentationCenter=""/>


<tags
	ms.service="documentdb"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="java"
	ms.topic="article"
	ms.date="09/20/2016"
	ms.author="anhoh"/>

#<a name="DocumentDB-HDInsight"></a>Ejecución de un trabajo de Hadoop con DocumentDB y HDInsight

Este tutorial muestra cómo ejecutar trabajos de [Apache Hive][apache-hive], [Apache Pig][apache-pig] y [Apache Hadoop][apache-hadoop] MapReduce en HDInsight de Azure con el conector Hadoop de DocumentDB. El conector de Hadoop de DocumentDB permite a DocumentDB actuar como punto de origen y recepción para trabajos de Hive, Pig y MapReduce. En este tutorial, se utilizará DocumentDB como el origen de datos y el destino para trabajos de Hadoop.

Después de completar este tutorial, podrá responder a las preguntas siguientes:

- ¿Cómo se cargan datos desde DocumentDB mediante un trabajo de MapReduce, Pig o Hive?
- ¿Cómo se almacenan datos en DocumentDB mediante un trabajo de MapReduce, Pig o Hive?

Se recomienda comenzar con el vídeo siguiente, donde se realiza una ejecución a través de un trabajo de Hive usando DocumentDB y HDInsight.

> [AZURE.VIDEO use-azure-documentdb-hadoop-connector-with-azure-hdinsight]

A continuación, vuelva a este artículo, donde recibirá información detallada sobre cómo ejecutar trabajos de análisis en los datos de DocumentDB.

> [AZURE.TIP] Este tutorial presupone que se tiene experiencia previa con Apache Hadoop, Hive o Pig Si no está familiarizado con Apache Hadoop, Hive y Pig, se recomienda consultar la [documentación de Apache Hadoop][apache-hadoop-doc]. Asimismo, el tutorial también presupone que se tiene experiencia previa con DocumentDB además de una cuenta en este servicio. Si no está familiarizado con DocumentDB o no tiene una cuenta en este servicio, consulte nuestra página [Introducción][getting-started].

¿No tiene tiempo para completar el tutorial y solo desea obtener todos los scripts de PowerShell de ejemplo de Hive, Pig y MapReduce? No hay problema, obténgalos [aquí][documentdb-hdinsight-samples]. La descarga también contiene los archivos hpl, pig y java para estos ejemplos.

## <a name="NewestVersion"></a>Versión más reciente

<table border='1'>
	<tr><th>Versión del conector de Hadoop</th>
		<td>1.2.0</td></tr>
	<tr><th>URI de script</th>
		<td>https://portalcontent.blob.core.windows.net/scriptaction/documentdb-hadoop-installer-v04.ps1</td></tr>
	<tr><th>Fecha de modificación</th>
		<td>26/04/2016</td></tr>
	<tr><th>Versiones compatibles de HDInsight</th>
		<td>3.1, 3.2</td></tr>
	<tr><th>Registro de cambios</th>
		<td>SDK de Java de DocumentDB actualizado a la versión 1.6.0.</br>
			Se ha agregado compatibilidad con las colecciones divididas como origen y receptor.</br>
		</td></tr>
</table>

## <a name="Prerequisites"></a>Requisitos previos
Antes de seguir las instrucciones de este tutorial, asegúrese de contar con lo siguiente:

- Una cuenta de DocumentDB, una base de datos y una colección con documentos dentro. Para obtener más información, consulte [Introducción a DocumentDB][getting-started]. Importe datos de ejemplo a su cuenta de DocumentDB con la [herramienta de importación de DocumentDB][documentdb-import-data].
- Capacidad de proceso. Las lecturas y escrituras de HDInsight se tienen en cuenta a la hora de calcular las unidades de solicitud asignadas a las colecciones. Para obtener más información, consulte [Operaciones de capacidad de proceso aprovisionada, unidades de solicitud y base de datos][documentdb-manage-throughput].
- Capacidad para un procedimiento almacenado adicional dentro de cada colección de salida. Los procedimientos almacenados se utilizan para transferir los documentos resultantes. Para obtener más información, consulte [Colecciones y rendimiento aprovisionado][documentdb-manage-document-storage].
- Capacidad para los documentos resultantes desde los trabajos de MapReduce, Pig o Hive. Para obtener más información, consulte [Gestión de la capacidad y rendimiento de DocumentDB][documentdb-manage-collections].
- [*Opcional*] Capacidad para una colección adicional. Para obtener más información, consulte [Almacenamiento de documentos aprovisionado y sobrecarga de índice][documentdb-manage-document-storage].

> [AZURE.WARNING] Para evitar que se cree una nueva colección mientras se está realizando cualquier trabajo, puede imprimir los resultados en stdout, guardar la salida en el contenedor WASB o especificar una colección existente. En el caso de que desee especificar una colección existente, se crearán nuevos documentos dentro de la colección y los documentos existentes solo se verán afectados si se produce un conflicto entre los *identificadores*. **El conector sobrescribirá automáticamente los documentos existentes con conflictos de identificador**. Puede desactivar esta característica estableciendo la opción upsert en false. Si upsert es false y se produce un conflicto, se producirá un error en el trabajo de Hadoop y se informará de un error a causa de conflicto de identificadores.


## <a name="ProvisionHDInsight"></a>Paso 1: Creación de un nuevo clúster de HDInsight
En este tutorial, se usa la acción de script de Azure Portal para personalizar el clúster de HDInsight. Así que usaremos Azure Portal para crear el clúster de HDInsight. Para obtener instrucciones sobre cómo usar los cmdlets de PowerShell o el SDK de .NET de HDInsight, consulte el artículo [Personalizar los clústeres de HDInsight mediante la acción de script][hdinsight-custom-provision].

1. Inicie sesión en el [Portal de Azure][azure-portal].

2. Haga clic en **+ Nuevo** en la parte superior del panel izquierdo y busque **HDInsight** en la barra de búsqueda superior de la hoja Nuevo.

3. Aparece **HDInsight** publicado por **Microsoft** en la parte superior de los resultados. Haga clic en él y luego en **rear**.

4. En la hoja para crear un nuevo clúster de HDInsight, escriba su **nombre de clúster** y seleccione la **suscripción** con la que quiere aprovisionar este recurso.

	<table border='1'>
		<tr><td>Nombre del clúster</td><td>Dé un nombre al clúster.<br/>
		Nombre DNS debe empezar y terminar con un carácter alfanumérico y puede contener guiones.<br/>
		El campo debe ser una cadena con una longitud que tenga entre 3 y 63 caracteres.</td></tr>
		<tr><td>Nombre de la suscripción</td>
			<td>Si tiene más de una suscripción de Azure, seleccione aquella que va a hospedar el clúster de HDInsight. </td></tr>
	</table>

5. Haga clic en **Seleccionar tipo de clúster** y establezca las siguientes propiedades en los valores especificados.

	<table border='1'>
		<tr><td>Tipo de clúster</td><td><strong>Hadoop</strong></td></tr>
		<tr><td>Nivel de clúster</td><td><strong>Estándar</strong></td></tr>
		<tr><td>Sistema operativo</td><td><strong>Windows</strong></td></tr>
		<tr><td>Versión</td><td>Versión más reciente</td></tr>
	</table>

	Ahora, haga clic en **SELECCIONAR**.

	![Proporcionar detalles del clúster inicial de HDInsight de Hadoop][image-customprovision-page1]

6. Haga clic en **Credenciales** para establecer las credenciales de inicio de sesión y de acceso remoto. Elija su **nombre de usuario de inicio de sesión del clúster** y **contraseña de inicio de sesión del clúster**.

	Si quiere tener acceso remoto al clúster, seleccione *Sí* en la parte inferior de la hoja y proporcione un nombre de usuario y una contraseña.

7. Haga clic en **rigen de datos** para establecer la ubicación principal para el acceso a datos. Elija el **método de selección** y especifique una cuenta de almacenamiento ya existente o cree una nueva.

8. En la misma hoja, especifique un **contenedor predeterminado** y una **ubicación**. A continuación, haga clic en **SELECCIONAR**.

	> [AZURE.NOTE] Seleccione una ubicación cercana a la región de la cuenta de DocumentDB para mejorar el rendimiento.

8. Haga clic en **recios** para seleccionar el número y el tipo de los nodos. Puede mantener la configuración predeterminada y escalar el número de nodos de trabajo más adelante.

9. Haga clic en **Configuración opcional** y luego en **Acciones de script** en la hoja Configuración opcional.

	En Acciones de Script, escriba la siguiente información para personalizar el clúster de HDInsight.

	<table border='1'>
		<tr><th>Propiedad</th><th>Valor</th></tr>
		<tr><td>Nombre</td>
			<td>Especifique un nombre para la acción de script.</td></tr>
		<tr><td>Identificador URI de script</td>
			<td>Especifique el identificador URI al script que se ha invocado para personalizar el clúster.</br></br>
			Especifique esta información: </br> <strong>https://portalcontent.blob.core.windows.net/scriptaction/documentdb-hadoop-installer-v04.ps1</strong>.</td></tr>
		<tr><td>Principal</td>
			<td>Haga clic en la casilla para ejecutar el script de PowerShell en el nodo principal.</br></br>
			<strong>Active esta casilla</strong>.</td></tr>
		<tr><td>Trabajo</td>
			<td>Haga clic en la casilla para ejecutar el script de PowerShell en el nodo de trabajo.</br></br>
			<strong>Active esta casilla</strong>.</td></tr>
		<tr><td>Zookeeper</td>
			<td>Haga clic en la casilla para ejecutar el script de PowerShell en el Zookeeper.</br></br>
			<strong>No es necesario</strong>.
			</td></tr>
		<tr><td>Parámetros</td>
			<td>Especifique los parámetros, si lo requiere el script.</br></br>
			<strong>No se necesita ningún parámetro</strong>.</td></tr>
	</table>

10. Cree un nuevo **grupo de recursos** o use uno existente en su suscripción de Azure.

11. Ahora, marque **Anclar al panel** para realizar un seguimiento de su implementación y haga clic en **Crear**.

## <a name="InstallCmdlets"></a>Paso 2: Instalación y configuración de Azure PowerShell

1. Instale Azure PowerShell. Puede encontrar instrucciones [aquí][powershell-install-configure].

	> [AZURE.NOTE] En el caso de las consultas de Hive, use el Editor de Hive en línea de HDInsight. Para ello, inicie sesión en [Azure Portal][azure-portal] y haga clic en **HDInsight** en el panel izquierdo para ver una lista de los clústeres de HDInsight. Haga clic en el clúster en el que desea ejecutar consultas de Hive y, a continuación, haga clic en **Consola de consultas**.

2. Abra el entorno de script integrado de Azure PowerShell:
	- Si el equipo dispone de Windows 8 o Windows Server 2012, o una versión posterior, puede utilizar la búsqueda integrada. En la pantalla de inicio, escriba **powershell ise** y haga clic en **Entrar**.
	- Si el equipo dispone de una versión anterior a Windows 8 o Windows Server 2012, use el menú Inicio. En el menú Inicio, escriba **Símbolo del sistema** en el cuadro de búsqueda. A continuación, en la lista de resultados, haga clic en **Símbolo del sistema**. En el Símbolo del sistema, escriba **powershell\_ise** y haga clic en **Entrar**.

3. Agregue su cuenta de Azure.
	1. En el panel de consola, escriba **Add-AzureAccount** y haga clic en **Entrar**.
	2. Escriba la dirección de correo electrónico asociada a su suscripción de Azure y haga clic en **Continuar**.
	3. Escriba la contraseña para la suscripción de Azure.
	4. Haga clic en **Iniciar sesión**.

4. El diagrama siguiente identifica las partes importantes de su entorno de scripts de Azure PowerShell.

	![Diagrama de Azure PowerShell][azure-powershell-diagram]

## <a name="RunHive"></a>Paso 3: Ejecución de un trabajo de Hive con DocumentDB y HDInsight

> [AZURE.IMPORTANT] Todas las variables indicadas con < > deben rellenarse con los valores de configuración adecuados.

1. Configure las siguientes variables en el panel de scripts de PowerShell.

		# Provide Azure subscription name, the Azure Storage account and container that is used for the default HDInsight file system.
		$subscriptionName = "<SubscriptionName>"
		$storageAccountName = "<AzureStorageAccountName>"
		$containerName = "<AzureStorageContainerName>"

		# Provide the HDInsight cluster name where you want to run the Hive job.
		$clusterName = "<HDInsightClusterName>"

2. <p>Comencemos a construir la cadena de consulta. Escribiremos una consulta de Hive que adopta las marcas de tiempo (_ts) de todos los documentos generados por el sistema y los identificadores únicos (_rid) de una colección de DocumentDB. Además, esta consulta registra todos los documentos por minuto y almacena de nuevo los resultados en una nueva colección de DocumentDB.</p>

    <p>En primer lugar, vamos a crear una tabla de Hive a partir de nuestra colección de DocumentDB. Agregue el siguiente fragmento de código en el panel de scripts de PowerShell <strong>después</strong> del fragmento de código número 1. Asegúrese de incluir el parámetro DocumentDB.query opcional para recortar nuestros documentos solo hasta _ts y _rid.</p>

    > [AZURE.NOTE] **La denominación de DocumentDB.inputCollections no era un error.** Sí, se pueden agregar varias colecciones como una entrada:</br>

		'*DocumentDB.inputCollections*' = '*<DocumentDB Input Collection Name 1>*,*<DocumentDB Input Collection Name 2>*' A1A</br> The collection names are separated without spaces, using only a single comma.


		# Create a Hive table using data from DocumentDB. Pass DocumentDB the query to filter transferred data to _rid and _ts.
		$queryStringPart1 = "drop table DocumentDB_timestamps; "  +
                            "create external table DocumentDB_timestamps(id string, ts BIGINT) "  +
                            "stored by 'com.microsoft.azure.documentdb.hive.DocumentDBStorageHandler' "  +
                            "tblproperties ( " +
                                "'DocumentDB.endpoint' = '<DocumentDB Endpoint>', " +
                                "'DocumentDB.key' = '<DocumentDB Primary Key>', " +
                                "'DocumentDB.db' = '<DocumentDB Database Name>', " +
                                "'DocumentDB.inputCollections' = '<DocumentDB Input Collection Name>', " +
                                "'DocumentDB.query' = 'SELECT r._rid AS id, r._ts AS ts FROM root r' ); "

3.  A continuación, vamos a crear una tabla de Hive para la colección de salida. Las propiedades del documento de salida serán el mes, el día, la hora, el minuto y el número total de apariciones.

	> [AZURE.NOTE] **Una vez más, la denominación de DocumentDB.outputCollections no era un error.** Sí, se pueden agregar varias colecciones como una salida: </br> 
	'*DocumentDB.outputCollections*' = '*\<DocumentDB Output Collection Name 1\>*,*\<DocumentDB Output Collection Name 2\>*' </br> Los nombres de las colecciones se separan sin espacios en blanco, solo con una coma. </br></br> 
	Documentos se distribuirán en cadena en varias colecciones. Un lote de documentos se almacenará en una colección. A continuación, un segundo lote de documentos se almacenará en la colección siguiente y así sucesivamente.

		# Create a Hive table for the output data to DocumentDB.
	    $queryStringPart2 = "drop table DocumentDB_analytics; " +
                              "create external table DocumentDB_analytics(Month INT, Day INT, Hour INT, Minute INT, Total INT) " +
                              "stored by 'com.microsoft.azure.documentdb.hive.DocumentDBStorageHandler' " +
                              "tblproperties ( " +
                                  "'DocumentDB.endpoint' = '<DocumentDB Endpoint>', " +
                                  "'DocumentDB.key' = '<DocumentDB Primary Key>', " +  
                                  "'DocumentDB.db' = '<DocumentDB Database Name>', " +
                                  "'DocumentDB.outputCollections' = '<DocumentDB Output Collection Name>' ); "

4. Por último, vamos realizar un recuento de los documentos por mes, día, hora y minuto, y vamos a insertar los resultados en la tabla de Hive de salida.

		# GROUP BY minute, COUNT entries for each, INSERT INTO output Hive table.
        $queryStringPart3 = "INSERT INTO table DocumentDB_analytics " +
                              "SELECT month(from_unixtime(ts)) as Month, day(from_unixtime(ts)) as Day, " +
                              "hour(from_unixtime(ts)) as Hour, minute(from_unixtime(ts)) as Minute, " +
                              "COUNT(*) AS Total " +
                              "FROM DocumentDB_timestamps " +
                              "GROUP BY month(from_unixtime(ts)), day(from_unixtime(ts)), " +
                              "hour(from_unixtime(ts)) , minute(from_unixtime(ts)); "

5. Agregue el siguiente fragmento de script para crear una definición de trabajo de Hive a partir de la consulta anterior.

		# Create a Hive job definition.
		$queryString = $queryStringPart1 + $queryStringPart2 + $queryStringPart3
		$hiveJobDefinition = New-AzureHDInsightHiveJobDefinition -Query $queryString

	También usará el conmutador -File para especificar un archivo de script de HiveQL en HDFS.

6. Agregue el siguiente fragmento para guardar la hora de inicio y enviar el trabajo de Hive.

		# Save the start time and submit the job to the cluster.
		$startTime = Get-Date
		Select-AzureSubscription $subscriptionName
		$hiveJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $hiveJobDefinition

7. Agregue lo siguiente para esperar a que el trabajo de Hive termine.

		# Wait for the Hive job to complete.
		Wait-AzureHDInsightJob -Job $hiveJob -WaitTimeoutInSeconds 3600

8. Agregue lo siguiente para imprimir la salida estándar y los tiempos de inicio y finalización.

		# Print the standard error, the standard output of the Hive job, and the start and end time.
		$endTime = Get-Date
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $hiveJob.JobId -StandardOutput
		Write-Host "Start: " $startTime ", End: " $endTime -ForegroundColor Green

9. **Ejecute** el nuevo script. **Haga clic** en el botón verde para llevar a cabo la ejecución.

10. Compruebe los resultados. Inicie sesión en el [Portal de Azure][azure-portal].
	1. Haga clic en <strong>Examinar</strong> en el panel de la izquierda. </br>
	2. Haga clic en <strong>Todo</strong>, en la parte superior derecha del panel de exploración. </br>
	3. Busque y haga clic en <strong>Cuentas de DocumentDB</strong>. </br>
	4. A continuación, busque su <strong>cuenta de DocumentDB</strong>, luego, la <strong>Base de datos de DocumentDB</strong> y su <strong>colección de DocumentDB</strong> asociadas a la colección de salida especificada en la consulta de Hive.</br>
	5. Por último, haga clic en <strong>Explorador de documentos</strong> debajo de <strong>Herramientas de desarrollo</strong>.</br></p>

	Verá los resultados de la consulta de Hive.

	![Resultados de la consulta de Hive][image-hive-query-results]

## <a name="RunPig"></a>Paso 4: Ejecución de un trabajo de Pig con DocumentDB y HDInsight

> [AZURE.IMPORTANT] Todas las variables indicadas con < > deben rellenarse con los valores de configuración adecuados.

1. Configure las siguientes variables en el panel de scripts de PowerShell.

        # Provide Azure subscription name.
        $subscriptionName = "Azure Subscription Name"

        # Provide HDInsight cluster name where you want to run the Pig job.
        $clusterName = "Azure HDInsight Cluster Name"

2. <p>Comencemos a construir la cadena de consulta. Escribiremos una consulta de Pig que adopta las marcas de tiempo (_ts) de todos los documentos generados por el sistema y los identificadores únicos (_rid) de una colección de DocumentDB. Además, esta consulta registra todos los documentos por minuto y almacena de nuevo los resultados en una nueva colección de DocumentDB.</p>
    <p>En primer lugar, cargue documentos de DocumentDB en HDInsight. Agregue el siguiente fragmento de código en el panel de scripts de PowerShell <strong>después</strong> del fragmento de código número 1. Asegúrese de agregar una consulta de DocumentDB para el parámetro de consulta de DocumentDB opcional para recortar los documentos a solo _ts y _rid.</p>
    
    > [AZURE.NOTE] Sí, permitir agregar varias colecciones como entrada: </br> 
    '*\<DocumentDB Input Collection Name 1\>*,*\<DocumentDB Input Collection Name 2\>*'</br> Se separan los nombres de la colección sin espacios en blanco, con una sola coma. </b>

	Documentos se distribuirán en cadena en varias colecciones. Un lote de documentos se almacenará en una colección. A continuación, un segundo lote de documentos se almacenará en la colección siguiente y así sucesivamente.

		# Load data from DocumentDB. Pass DocumentDB query to filter transferred data to _rid and _ts.
        $queryStringPart1 = "DocumentDB_timestamps = LOAD '<DocumentDB Endpoint>' USING com.microsoft.azure.documentdb.pig.DocumentDBLoader( " +
                                                        "'<DocumentDB Primary Key>', " +
                                                        "'<DocumentDB Database Name>', " +
                                                        "'<DocumentDB Input Collection Name>', " +
                                                        "'SELECT r._rid AS id, r._ts AS ts FROM root r' ); "

3.  A continuación, vamos realizar un recuento de los documentos por mes, día, hora, minuto y el número total de apariciones.

		# GROUP BY minute and COUNT entries for each.
        $queryStringPart2 = "timestamp_record = FOREACH DocumentDB_timestamps GENERATE `$0#'id' as id:int, ToDate((long)(`$0#'ts') * 1000) as timestamp:datetime; " +
                            "by_minute = GROUP timestamp_record BY (GetYear(timestamp), GetMonth(timestamp), GetDay(timestamp), GetHour(timestamp), GetMinute(timestamp)); " +
                            "by_minute_count = FOREACH by_minute GENERATE FLATTEN(group) as (Year:int, Month:int, Day:int, Hour:int, Minute:int), COUNT(timestamp_record) as Total:int; "

4. Por último, vamos a almacenar los resultados en nuestra nueva colección de salida.

    > [AZURE.NOTE] Sí, permitir agregar varias colecciones como salida:</br>
    '\<DocumentDB Output Collection Name 1\>,\<DocumentDB Output Collection Name 2\>'</br> Se separan los nombres de la colección sin espacios en blanco, con una sola coma.</br>
    Documentos se distribuirán en cadena por las distintas colecciones. Un lote de documentos se almacenará en una colección. A continuación, un segundo lote de documentos se almacenará en la colección siguiente y así sucesivamente.

		# Store output data to DocumentDB.
        $queryStringPart3 = "STORE by_minute_count INTO '<DocumentDB Endpoint>' " +
                            "USING com.microsoft.azure.documentdb.pig.DocumentDBStorage( " +
                                "'<DocumentDB Primary Key>', " +
                                "'<DocumentDB Database Name>', " +
                                "'<DocumentDB Output Collection Name>'); "

5. Agregue el siguiente fragmento de script para crear una definición de trabajo de Pig a partir de la consulta anterior.

        # Create a Pig job definition.
        $queryString = $queryStringPart1 + $queryStringPart2 + $queryStringPart3
        $pigJobDefinition = New-AzureHDInsightPigJobDefinition -Query $queryString -StatusFolder $statusFolder

	También usará el modificador -File para especificar un archivo de script de Pig en HDFS.

6. Agregue el siguiente fragmento para guardar la hora de inicio y enviar el trabajo de Pig.

		# Save the start time and submit the job to the cluster.
		$startTime = Get-Date
		Select-AzureSubscription $subscriptionName
		$pigJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $pigJobDefinition

7. Agregue lo siguiente para esperar a que el trabajo de Pig termine.

		# Wait for the Pig job to complete.
		Wait-AzureHDInsightJob -Job $pigJob -WaitTimeoutInSeconds 3600

8. Agregue lo siguiente para imprimir la salida estándar y los tiempos de inicio y finalización.

		# Print the standard error, the standard output of the Hive job, and the start and end time.
		$endTime = Get-Date
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $pigJob.JobId -StandardOutput
		Write-Host "Start: " $startTime ", End: " $endTime -ForegroundColor Green

9. **Ejecute** el nuevo script. **Haga clic** en el botón verde para llevar a cabo la ejecución.

10. Compruebe los resultados. Inicie sesión en el [Portal de Azure][azure-portal].
	1. Haga clic en <strong>Examinar</strong> en el panel de la izquierda. </br>
	2. Haga clic en <strong>Todo</strong>, en la parte superior derecha del panel de exploración. </br>
	3. Busque y haga clic en <strong>Cuentas de DocumentDB</strong>. </br>
	4. A continuación, busque su <strong>cuenta de DocumentDB</strong>, después, la <strong>base de datos de DocumentDB</strong> y su <strong>colección de DocumentDB</strong> asociadas a la colección de salida especificada en la consulta de Pig.</br>
	5. Por último, haga clic en <strong>Explorador de documentos</strong> debajo de <strong>Herramientas de desarrollo</strong>.</br></p>

	Verá los resultados de la consulta de Pig.

	![Resultados de la consulta de Pig][image-pig-query-results]

## <a name="RunMapReduce"></a>Paso 5: Ejecución de un trabajo de MapReduce con DocumentDB y HDInsight

1. Configure las siguientes variables en el panel de scripts de PowerShell.

		$subscriptionName = "<SubscriptionName>"   # Azure subscription name
		$clusterName = "<ClusterName>"             # HDInsight cluster name

2. Ejecutaremos un trabajo de MapReduce que cuenta el número de repeticiones de cada propiedad de documento de su colección de DocumentDB. Agregue este fragmento de script **después** del fragmento de código anterior.

		# Define the MapReduce job.
		$TallyPropertiesJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/TallyProperties-v01.jar" -ClassName "TallyProperties" -Arguments "<DocumentDB Endpoint>","<DocumentDB Primary Key>", "<DocumentDB Database Name>","<DocumentDB Input Collection Name>","<DocumentDB Output Collection Name>","<[Optional] DocumentDB Query>"

	> [AZURE.NOTE] TallyProperties-v01.jar incluye la instalación personalizada del conector de Hadoop de DocumentDB.

3. Agregue el siguiente comando para enviar el trabajo de MapReduce.

		# Save the start time and submit the job.
		$startTime = Get-Date
		Select-AzureSubscription $subscriptionName
		$TallyPropertiesJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $TallyPropertiesJobDefinition | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600  

	Además de la definición de trabajo de MapReduce, también debe proporcionar el nombre del clúster de HDInsight en el que desea ejecutar el trabajo de MapReduce y las credenciales. Start-AzureHDInsightJob es una llamada no sincronizada. Para comprobar la finalización del trabajo, use el cmdlet *Wait-AzureHDInsightJob*.

4. Agregue el siguiente comando para comprobar posibles errores al ejecutar el trabajo de MapReduce.

		# Get the job output and print the start and end time.
		$endTime = Get-Date
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $TallyPropertiesJob.JobId -StandardError
		Write-Host "Start: " $startTime ", End: " $endTime -ForegroundColor Green

5. **Ejecute** el nuevo script. **Haga clic** en el botón verde para llevar a cabo la ejecución.

6. Compruebe los resultados. Inicie sesión en el [Portal de Azure][azure-portal].
	1. Haga clic en <strong>Examinar</strong> en el panel de la izquierda.
	2. Haga clic en <strong>Todo</strong>, en la parte superior derecha del panel de exploración.
	3. Busque y haga clic en <strong>Cuentas de DocumentDB</strong>.
	4. A continuación, busque su <strong>Cuenta de DocumentDB</strong>, a continuación, la <strong>Base de datos de DocumentDB</strong> y su <strong>Colección de DocumentDB</strong> asociadas a la colección de salida especificada en el trabajo de MapReduce.
	5. Por último, haga clic en <strong>Explorador de documentos</strong>, debajo de <strong>Herramientas de desarrollo</strong>.

	Verá los resultados de su trabajo de MapReduce.

	![Resultados de la consulta de MapReduce][image-mapreduce-query-results]

## <a name="NextSteps"></a>Pasos siguientes

¡Enhorabuena! Acaba de ejecutar sus primeros trabajos de Hive, Pig y MapReduce con Azure DocumentDB y HDInsight.

El conector de Hadoop tiene el código abierto. Si le interesa, puede contribuir en [GitHub][documentdb-github].

Para obtener más información, consulte los artículos siguientes:

- [Desarrollo de una aplicación Java con DocumentDB][documentdb-java-application]
- [Desarrollo de programas MapReduce de Java para Hadoop en HDInsight][hdinsight-develop-deploy-java-mapreduce]
- [Introducción al uso de Hadoop con Hive en HDInsight para analizar el uso de datos móviles][hdinsight-get-started]
- [Uso de MapReduce con HDInsight][hdinsight-use-mapreduce]
- [Uso de Hive con HDInsight][hdinsight-use-hive]
- [Uso de Pig con HDInsight][hdinsight-use-pig]
- [Personalizar los clústeres de HDInsight mediante la acción de script][hdinsight-hadoop-customize-cluster]

[apache-hadoop]: http://hadoop.apache.org/
[apache-hadoop-doc]: http://hadoop.apache.org/docs/current/
[apache-hive]: http://hive.apache.org/
[apache-pig]: http://pig.apache.org/
[getting-started]: documentdb-get-started.md

[azure-portal]: https://portal.azure.com/
[azure-powershell-diagram]: ./media/documentdb-run-hadoop-with-hdinsight/azurepowershell-diagram-med.png

[documentdb-hdinsight-samples]: http://portalcontent.blob.core.windows.net/samples/documentdb-hdinsight-samples.zip
[documentdb-github]: https://github.com/Azure/azure-documentdb-hadoop
[documentdb-java-application]: documentdb-java-application.md
[documentdb-manage-collections]: documentdb-manage.md#Collections
[documentdb-manage-document-storage]: documentdb-manage.md#IndexOverhead
[documentdb-manage-throughput]: documentdb-manage.md#ProvThroughput
[documentdb-import-data]: documentdb-import-data.md

[hdinsight-custom-provision]: ../hdinsight/hdinsight-provision-clusters.md#powershell
[hdinsight-develop-deploy-java-mapreduce]: ../hdinsight/hdinsight-develop-deploy-java-mapreduce-linux.md
[hdinsight-hadoop-customize-cluster]: ../hdinsight/hdinsight-hadoop-customize-cluster.md
[hdinsight-get-started]: ../hdinsight/hdinsight-hadoop-tutorial-get-started-windows.md
[hdinsight-storage]: ../hdinsight/hdinsight-hadoop-use-blob-storage.md
[hdinsight-use-hive]: ../hdinsight/hdinsight-use-hive.md
[hdinsight-use-mapreduce]: ../hdinsight/hdinsight-use-mapreduce.md
[hdinsight-use-pig]: ../hdinsight/hdinsight-use-pig.md

[image-customprovision-page1]: ./media/documentdb-run-hadoop-with-hdinsight/customprovision-page1.png
[image-hive-query-results]: ./media/documentdb-run-hadoop-with-hdinsight/hivequeryresults.PNG
[image-mapreduce-query-results]: ./media/documentdb-run-hadoop-with-hdinsight/mapreducequeryresults.PNG
[image-pig-query-results]: ./media/documentdb-run-hadoop-with-hdinsight/pigqueryresults.PNG

[powershell-install-configure]: ../powershell-install-configure.md

<!---HONumber=AcomDC_0921_2016-->