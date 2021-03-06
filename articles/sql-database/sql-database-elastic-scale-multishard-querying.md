<properties 
	pageTitle="Consultas a través de particiones múltiples | Microsoft Azure" 
	description="Ejecute consultas a través de particiones con la biblioteca de cliente de bases de datos elásticas." 
	services="sql-database" 
	documentationCenter="" 
	manager="jhubbard" 
	authors="torsteng" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.workload="sql-database" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/12/2016" 
	ms.author="torsteng"/>

# Consultas a través de particiones múltiples

## Información general

Con las [herramientas de Base de datos elástica](sql-database-elastic-scale-introduction.md), puede crear soluciones de base de datos particionadas. **Consultas a través de particiones múltiples** se usa para tareas como informes y recopilación de datos que requieren ejecutar una consulta que abarca varias particiones. (Compare esto con el [enrutamiento dependiente de los datos](sql-database-elastic-scale-data-dependent-routing.md), que ejecuta todo el trabajo en una sola partición).

## Información general

1. Obtenga una clase [**RangeShardMap**](https://msdn.microsoft.com/library/azure/dn807318.aspx) o [**ListShardMap**](https://msdn.microsoft.com/library/azure/dn807370.aspx) utilizando el método [**TryGetRangeShardMap**](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetrangeshardmap.aspx), [**TryGetListShardMap**](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetlistshardmap.aspx) o [**GetShardMap**](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.getshardmap.aspx). Consulte [**Construcción de un objeto ShardMapManager**](sql-database-elastic-scale-shard-map-management.md#constructing-a-shardmapmanager) y [**Obtención de una clase RangeShardMap o ListShardMap**](sql-database-elastic-scale-shard-map-management.md#get-a-rangeshardmap-or-listshardmap).
2. Cree un objeto **[MultiShardConnection](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.query.multishardconnection.aspx)**.
2. Cree un objeto **[MultiShardCommand](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand.aspx)**. 
3. Establezca la **[propiedad CommandText](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand.commandtext.aspx#P:Microsoft.Azure.SqlDatabase.ElasticScale.Query.MultiShardCommand.CommandText)** en un comando T-SQL.
3. Ejecute el comando mediante la invocación del **[método ExecuteReader](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand.executereader.aspx)**.
4. Vea los resultados con la **[clase MultiShardDataReader](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.query.multisharddatareader.aspx)**. 

## Ejemplo

El código siguiente ilustra el uso de consultas a través de particiones múltiples con un **ShardMap** llamado *myShardMap*.

    using (MultiShardConnection conn = new MultiShardConnection( 
                                        myShardMap.GetShards(), 
                                        myShardConnectionString) 
          ) 
    { 
    using (MultiShardCommand cmd = conn.CreateCommand())
           { 
            cmd.CommandText = "SELECT c1, c2, c3 FROM ShardedTable"; 
            cmd.CommandType = CommandType.Text; 
            cmd.ExecutionOptions = MultiShardExecutionOptions.IncludeShardNameColumn; 
            cmd.ExecutionPolicy = MultiShardExecutionPolicy.PartialResults; 

            using (MultiShardDataReader sdr = cmd.ExecuteReader()) 
            	{ 
                	while (sdr.Read())
                    	{ 
                        	var c1Field = sdr.GetString(0); 
                        	var c2Field = sdr.GetFieldValue<int>(1); 
                        	var c3Field = sdr.GetFieldValue<Int64>(2);
                    	} 
             	} 
           } 
    } 

 
Una diferencia clave es la construcción de conexiones entre particiones múltiples. Donde **SqlConnection** opera en una sola base de datos, **MultiShardConnection** toma una ***colección de particiones*** como entrada. Complete la recopilación de particiones a partir de un mapa de particiones. Luego la consulta se ejecuta en la recopilación de particiones usando la semántica **UNION ALL** para ensamblar un solo resultado global. De manera opcional, el nombre de la partición donde se origina la fila se puede agregar a la salida usando la propiedad **ExecutionOptions** en el comando.

Observe la llamada a **myShardMap.GetShards()**. Este método recupera todas las particiones desde el mapa de particiones y brinda una manera fácil de ejecutar una consulta a través de todas las bases de datos importantes. La colección de particiones para una consulta a través de particiones múltiples se puede restringir aún más si se realiza una consulta LINQ sobre la colección devuelta desde la llamada a **myShardMap.GetShards()**. En combinación con la directiva de resultados parciales, la actual capacidad en las consultas a través de particiones múltiples se diseñó para funcionar bien con decenas, y hasta centenas, de particiones.

Una limitación con las consultas a través de particiones múltiples actualmente es la falta de validación de las particiones y de los shardlets que se consultan. Mientras que el enrutamiento dependiente de los datos comprueba que una partición determinada sea parte del mapa de particiones en el momento de la consulta, las consultas a través de particiones múltiples no realizan esta comprobación. Esto puede llevar a que las consultas a través de particiones múltiples se ejecuten en bases de datos que se han quitado del mapa de particiones.

## Consultas a través de particiones múltiples y operaciones de división y combinación

Las consultas a través de particiones múltiples no comprueban si los shardlets en la base de datos consultada forman parte de las operaciones de división y combinación en curso. (Consulte [Escalado con la herramienta de división y combinación de Base de datos elástica](sql-database-elastic-scale-overview-split-and-merge.md)). Esto puede llevar a incoherencias donde las filas del mismo shardlet se muestran para bases de datos múltiples en la misma consulta a través de particiones múltiples. Tenga en cuenta estas limitaciones y considere la posibilidad de agotar las operaciones de división y combinación y los cambios en el mapa de particiones mientras se realizan consultas a través de particiones múltiples.

[AZURE.INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

## Consulte también
Clases y métodos **[System.Data.SqlClient](http://msdn.microsoft.com/library/System.Data.SqlClient.aspx)**.


Administre las particiones con la [biblioteca de cliente de Base de datos elástica](sql-database-elastic-database-client-library.md). Incluye nuevo espacio de nombres llamado [Microsoft.Azure.SqlDatabase.ElasticScale.Query](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.query.aspx), el que proporciona la capacidad de consultar varias particiones usando una sola solicitud y resultado. Brinda una abstracción de consulta sobre una recopilación de particiones. También proporciona directivas de ejecución alternativas, en especial resultados parciales, para abordar errores cuando se consulte sobre muchas particiones.

 

<!---HONumber=AcomDC_0420_2016-->