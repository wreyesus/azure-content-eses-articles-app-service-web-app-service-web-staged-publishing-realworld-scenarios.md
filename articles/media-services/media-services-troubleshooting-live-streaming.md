<properties 
	pageTitle="Guía de solución de problemas para el streaming en vivo" 
	description="En este tema se ofrecen sugerencias sobre cómo solucionar problemas del streaming en vivo." 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/22/2016"  
	ms.author="juliako"/>

#Guía de solución de problemas para el streaming en vivo

En este tema se ofrecen sugerencias sobre cómo solucionar algunos problemas del streaming en vivo.

## Problemas relacionados con los codificadores locales 

Esta sección se ofrecen sugerencias sobre cómo solucionar problemas relacionados con codificadores locales que están configurados para enviar una secuencia de velocidad de bits única a los canales de AMS que están habilitados para la codificación en directo.

###Problema: le gustaría ver registros 

- **Posible problema**: no puede encontrar los registros del codificador que podrían ayudar en los problemas de depuración.
	
	- **Telestream Wirecast**: normalmente encontrará los registros en C:\\Users{username}\\AppData\\Roaming\\Wirecast\\
	- **Elemental Live**: puede encontrar vínculos a los registros en el Portal de administración. Haga clic en **Estadísticas** y luego en **Registros**. En la página **Log Files** (Archivos de registro), verá una lista de registros para todos los elementos de LiveEvent; seleccione el que coincida con su sesión actual.
	- **Flash Media Encoder Live**: puede encontrar el **directorio de registro** desplazándose hasta la pestaña **Encoding Log** (Registro de codificación).
	
###Problema: no hay ninguna opción para generar una secuencia progresiva

- **Posible problema**: el codificador que se usa no deshace el entrelazado automáticamente.

	**Pasos para solucionar problemas**: busque una opción de eliminación de entrelazado dentro de la interfaz del codificador. Cuando se habilite la opción de eliminación de entrelazado, compruebe de nuevo la configuración de la salida progresiva.
 
###Problema: se intentaron varias configuraciones de salida del codificador y todavía no es posible conectarse. 

- **Posible problema**: el canal de codificación Azure no se restableció correctamente.

	**Pasos para solucionar problemas**: asegúrese de que el codificador ya no está insertando en AMS, detenga y restablezca el canal. Cuando se vuelva a ejecutar, intente conectar su codificador con la nueva configuración. Si esto sigue sin solucionar el problema, intente crear un nuevo canal por completo; a veces, los canales se pueden dañar tras varios intentos con error.

- **Posible problema**: la configuración del fotograma clave o el tamaño de GOP no son óptimos.

	**Pasos para solucionar problemas**: el intervalo de fotogramas clave o el tamaño de GOP recomendado es de 2 segundos. Algunos codificadores calculan esta configuración en el número de fotogramas, mientras que otros usan segundos. Por ejemplo: cuando se envían 30 fps, el tamaño de GOP sería de 60 fotogramas, lo que equivale a 2 segundos.
	 
- **Posible problema**: los puertos cerrados están bloqueando la secuencia.

	**Pasos para solucionar problemas**: al hacer el streaming mediante RTMP, compruebe la configuración del firewall o del proxy para confirmar que los puertos salientes 1935 y 1936 están abiertos. Cuando se use el streaming de RTP, confirme que el puerto saliente 2010 está abierto.


###Problema: al configurar el codificador para que haga streaming con el protocolo RTP, no hay ningún lugar para escribir un nombre de host. 

- **Posible problema**: muchos codificadores de RTP no permiten nombres de host y se necesitará adquirir una dirección IP.

	**Pasos para solucionar problemas**: para buscar la dirección IP, abra un símbolo del sistema en cualquier equipo. Para hacer esto en Windows, abra el selector de ejecución (WIN + R) y escriba "cmd" para abrir.

	Cuando se abra el símbolo del sistema, escriba "Ping [nombre de host de AMS]".

	El nombre de host se puede derivar omitiendo el número de puerto de la URL de introducción de Azure, como se resalta en el siguiente ejemplo:

	rtp://test2-amstest009.rtp.channel.mediaservices.windows.net:2010/

	![fmle](./media/media-services-fmle-live-encoder/media-services-fmle10.png)

###Problema: no se puede reproducir la secuencia publicada.
 
- **Posible problema**: no hay ningún extremo de streaming en ejecución o no hay ninguna unidad de streaming (unidades de escala) asignada.

	**Pasos para solucionar problemas**: vaya a la pestaña "Extremo de streaming" en la herramienta AMSE y compruebe que hay un extremo de streaming con una unidad de streaming.
	


>[AZURE.NOTE] Si después de seguir los pasos de solución de problemas, todavía no puede realizar correctamente la transmisión, envíe una incidencia de soporte técnico mediante el Portal de Azure clásico.

##Rutas de aprendizaje de Servicios multimedia

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##Envío de comentarios

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

<!---HONumber=AcomDC_0629_2016-->