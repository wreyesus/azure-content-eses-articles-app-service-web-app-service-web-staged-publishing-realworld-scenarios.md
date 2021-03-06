<properties
   pageTitle="Desarrollo de topologías basadas en Java para Apache Storm | Microsoft Azure"
   description="Aprenda a crear topologías de Storm en Java mediante la creación de una topología de recuento de palabras sencilla."
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="jhubbard"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
   ms.service="hdinsight"
   ms.devlang="java"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="09/14/2016"
   ms.author="larryfr"/>

#Desarrolle topologías basadas en Java para una aplicación básica de recuento de palabras con Apache Storm y Maven en HDInsight

Aprenda cómo crear una topología basada en Java para Apache Storm en HDInsight mediante Maven. Seguirá el proceso de creación de una aplicación básica de recuento de palabras mediante Maven y Java, con la topología definida en Java. Después, verá cómo definir la topología con el marco de trabajo Flux.

> [AZURE.NOTE] El marco de trabajo Flux está disponible en Storm 0.10.0 o versiones posteriores. Storm 0.10.0 está disponible con HDInsight 3.3 y 3.4.

Después de completar los pasos descritos en este documento, tendrá una topología básica que puede implementar en Apache Storm en HDInsight.

> [AZURE.NOTE] En [https://github.com/Azure-Samples/hdinsight-java-storm-wordcount](https://github.com/Azure-Samples/hdinsight-java-storm-wordcount) encontrará una versión completa de las topologías creadas en este documento.

##Requisitos previos

* <a href="https://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html" target="_blank">Kit de desarrolladores de Java (JDK) versión 7</a>

* <a href="https://maven.apache.org/download.cgi" target="_blank">Maven</a>: Maven es un sistema de compilación de proyectos para proyectos de Java.

* Un editor de texto como Bloc de notas, <a href="http://www.gnu.org/software/emacs/" target="_blank">Emacs<a>, <a href="http://www.sublimetext.com/" target="_blank">Sublime Text</a>, <a href="https://atom.io/" target="_blank">Atom.io</a>, <a href="http://brackets.io/" target="_blank">Brackets.io</a>. O puede usar un entorno de desarrollo integrado (IDE), como <a href="https://eclipse.org/" target="_blank">Eclipse</a> (versión Luna o posterior).

	> [AZURE.NOTE] El editor o IDE puede tener una funcionalidad específica para trabajar con Maven que no se trata en este documento. Para obtener información sobre las capacidades de su entorno de edición, consulte la documentación del producto que esté utilizando.

##Configuración de las variables de entorno

Pueden establecer las siguientes variables de entorno al instalar Java y el JDK. Sin embargo, debe comprobar que existen y que contienen los valores correctos para su sistema.

* **JAVA\_HOME**: debe apuntar al directorio donde está instalado Java Runtime Environment (JRE). Por ejemplo, en una distribución de Unix o Linux, debe tener un valor similar a `/usr/lib/jvm/java-7-oracle`. En Windows, tendría un valor similar a `c:\Program Files (x86)\Java\jre1.7`.

* **PATH**: debe contener las rutas de acceso siguientes:

	* **JAVA\_HOME** (o la ruta de acceso equivalente).

	* **JAVA\_HOME\\bin** (o la ruta de acceso equivalente).

	* El directorio donde está instalado Maven

##Creación de un nuevo proyecto de Maven

En la línea de comandos, use el siguiente código para crear un nuevo proyecto de Maven llamado **WordCount**:

	mvn archetype:generate -DarchetypeArtifactId=maven-archetype-quickstart -DgroupId=com.microsoft.example -DartifactId=WordCount -DinteractiveMode=false

Esto creará un nuevo directorio denominado **WordCount** en la ubicación actual, que contiene un proyecto de Maven básico.

El directorio **WordCount** contendrá los siguientes elementos:

* **pom.xml**: contiene la configuración del proyecto de Maven

* **src\\main\\java\\com\\microsoft\\example**: contiene el código de la aplicación

* **src\\test\\java\\com\\microsoft\\example**: contiene pruebas para la aplicación. En este ejemplo, no se crearán pruebas.

###Supresión del código de ejemplo

Dado que se creará una aplicación propia, elimine los archivos de la aplicación y las pruebas generadas:

*  **src\\test\\java\\com\\microsoft\\example\\AppTest.java**

*  **src\\main\\java\\com\\microsoft\\example\\App.java**

##Agregar propiedades

Maven permite definir los valores de nivel de proyecto llamados propiedades. Agregue lo siguiente después de la línea `<url>http://maven.apache.org</url>`:

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <!--
        Storm 0.10.0 is for HDInsight 3.3 and 3.4.
        To find the version information for earlier HDInsight cluster
        versions, see https://azure.microsoft.com/es-ES/documentation/articles/hdinsight-component-versioning/
        -->
        <storm.version>0.10.0</storm.version>
    </properties>

Ahora podemos usar estos valores en otras secciones. Por ejemplo, al especificar la versión de los componentes de Storm, podemos usar `${storm.version}` en lugar de codificar un valor de forma rígida.

##Adición de dependencias

Dado que se trata de una topología de Storm, debe agregar una dependencia para los componentes de Storm. Abra el archivo **pom.xml** y agregue el código siguiente en la sección **&lt;dependencias>**.

	<dependency>
	  <groupId>org.apache.storm</groupId>
	  <artifactId>storm-core</artifactId>
      <version>${storm.version}</version>
	  <!-- keep storm out of the jar-with-dependencies -->
	  <scope>provided</scope>
	</dependency>

En tiempo de compilación, Maven usa esta información para buscar **storm-core** en el repositorio de Maven. Busca primero en el repositorio del equipo local. Si los archivos no están allí, se descargan desde el repositorio de Maven público y se almacenan en el repositorio local.

> [AZURE.NOTE] Observe la línea `<scope>provided</scope>` en la sección que se agregó. Indica a Maven que excluya **storm-core** de los archivos JAR que creemos, ya que el sistema lo proporcionará. Esto permite que los paquetes creados sean un poco más pequeños y se garantiza que usarán los bits de **storm-core** incluidos en el clúster de Storm en HDInsight.

##Configuración de compilación

Los complementos de Maven le permiten personalizar las fases de la compilación del proyecto, por ejemplo, cómo se compila el proyecto y cómo se empaqueta en un archivo JAR. Abra el archivo **pom.xml** y agregue el código siguiente directamente encima de la línea `</project>`.

	<build>
	  <plugins>
	  </plugins>
      <resources>
      </resources>
	</build>

Esta sección se utilizará para agregar complementos, recursos y otras opciones de configuración de compilación. Para obtener una referencia completa del archivo __pom.xml__, consulte [http://maven.apache.org/pom.html](http://maven.apache.org/pom.html).

###Agregar complementos

En el caso de las topologías de Storm, el <a href="http://mojo.codehaus.org/exec-maven-plugin/" target="_blank">Complemento Exec Maven</a> es útil porque le permite ejecutar fácilmente la topología de manera local en su entorno de desarrollo. Agregue lo siguiente a la sección `<plugins>` del archivo **pom.xml** para incluir el complemento Exec Maven:

	<plugin>
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>exec-maven-plugin</artifactId>
      <version>1.4.0</version>
      <executions>
        <execution>
        <goals>
          <goal>exec</goal>
        </goals>
        </execution>
      </executions>
      <configuration>
        <executable>java</executable>
        <includeProjectDependencies>true</includeProjectDependencies>
        <includePluginDependencies>false</includePluginDependencies>
        <classpathScope>compile</classpathScope>
        <mainClass>${storm.topology}</mainClass>
      </configuration>
    </plugin>

> [AZURE.NOTE] Tenga en cuenta que la entrada `<mainClass>` usa `${storm.topology}`. Esto no se definió anteriormente en la sección de propiedades (pero podríamos haberlo hecho). En su lugar, este valor se establecerá en la línea de comandos cuando se ejecute la topología en el entorno de desarrollo, en un paso posterior.

Otro complemento útil es el <a href="http://maven.apache.org/plugins/maven-compiler-plugin/" target="_blank">Complemento Apache Maven Compiler</a>, que se usa para cambiar las opciones de compilación. Es necesario principalmente para cambiar la versión de Java que usa Maven para el origen y destino de la aplicación. Queremos la versión 1.7.

Agregue lo siguiente en la sección `<plugins>` del archivo **pom.xml** para incluir el complemento Apache Maven Compiler y establezca las versiones de origen y de destino en 1.7.

	<plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.3</version>
      <configuration>
        <source>1.7</source>
        <target>1.7</target>
      </configuration>
    </plugin>

###Configure resources

Gracias a la sección de recursos, podrá incluir recursos no basados en códigos, como los archivos de configuración que necesitan los componentes de la topología. En este ejemplo, agregue lo siguiente en la sección `<resources>` del archivo **pom.xml**.

    <resource>
        <directory>${basedir}/resources</directory>
        <filtering>false</filtering>
        <includes>
          <include>log4j2.xml</include>
        </includes>
    </resource>

Con esto, se agrega el directorio de recursos en la raíz del proyecto (`${basedir}`) como una ubicación que contiene recursos e incluye el archivo denominado "__log4j2.xml__". Este archivo se utiliza para configurar la información que registra la topología.

##Creación de la topología

Una topología de Storm basada en Java consta de tres componentes que debe crear (o hacer referencia) como una dependencia.

* **Spouts**: lee datos de orígenes externos y emite flujos de datos a la topología.

* **Bolts**: realiza el procesamiento en flujos que emite spouts u otros bolts, y emite uno o varios flujos.

* **Topology**: define cómo se organizan los spouts y bolts, y proporciona el punto de entrada de la topología.

###Creación del spout

Para reducir los requisitos de la configuración de los orígenes de datos externos, el spout siguiente simplemente emite frases aleatorias. Se trata de una versión modificada de un spout que se proporciona con los [ejemplos de Storm-Starter](https://github.com/apache/storm/blob/0.10.x-branch/examples/storm-starter/src/jvm/storm/starter).

> [AZURE.NOTE] Para ver un ejemplo de un spout que lee desde un origen de datos externos, consulte uno de los siguientes ejemplos:
>
> * [TwitterSampleSpout](https://github.com/apache/storm/blob/0.10.x-branch/examples/storm-starter/src/jvm/storm/starter/spout/TwitterSampleSpout.java): un spout de ejemplo que lee desde Twitter.
>
> * [Storm Kafka](https://github.com/apache/storm/tree/0.10.x-branch/external/storm-kafka): un spout que lee desde Kafka.

Para el spout, cree un nuevo archivo denominado **RandomSentenceSpout.java** en el directorio **src\\main\\java\\com\\microsoft\\example** y use lo siguiente como contenido:

    /**
     * Licensed to the Apache Software Foundation (ASF) under one
     * or more contributor license agreements.  See the NOTICE file
     * distributed with this work for additional information
     * regarding copyright ownership.  The ASF licenses this file
     * to you under the Apache License, Version 2.0 (the
     * "License"); you may not use this file except in compliance
     * with the License.  You may obtain a copy of the License at
     *
     * http://www.apache.org/licenses/LICENSE-2.0
     *
     * Unless required by applicable law or agreed to in writing, software
     * distributed under the License is distributed on an "AS IS" BASIS,
     * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     * See the License for the specific language governing permissions and
     * limitations under the License.
     */

     /**
      * Original is available at https://github.com/apache/storm/blob/master/examples/storm-starter/src/jvm/storm/starter/spout/RandomSentenceSpout.java
      */

    package com.microsoft.example;

    import backtype.storm.spout.SpoutOutputCollector;
    import backtype.storm.task.TopologyContext;
    import backtype.storm.topology.OutputFieldsDeclarer;
    import backtype.storm.topology.base.BaseRichSpout;
    import backtype.storm.tuple.Fields;
    import backtype.storm.tuple.Values;
    import backtype.storm.utils.Utils;

    import java.util.Map;
    import java.util.Random;

    //This spout randomly emits sentences
    public class RandomSentenceSpout extends BaseRichSpout {
      //Collector used to emit output
      SpoutOutputCollector _collector;
      //Used to generate a random number
      Random _rand;

      //Open is called when an instance of the class is created
      @Override
      public void open(Map conf, TopologyContext context, SpoutOutputCollector collector) {
      //Set the instance collector to the one passed in
        _collector = collector;
        //For randomness
        _rand = new Random();
      }

      //Emit data to the stream
      @Override
      public void nextTuple() {
      //Sleep for a bit
        Utils.sleep(100);
        //The sentences that will be randomly emitted
        String[] sentences = new String[]{ "the cow jumped over the moon", "an apple a day keeps the doctor away",
            "four score and seven years ago", "snow white and the seven dwarfs", "i am at two with nature" };
        //Randomly pick a sentence
        String sentence = sentences[_rand.nextInt(sentences.length)];
        //Emit the sentence
        _collector.emit(new Values(sentence));
      }

      //Ack is not implemented since this is a basic example
      @Override
      public void ack(Object id) {
      }

      //Fail is not implemented since this is a basic example
      @Override
      public void fail(Object id) {
      }

      //Declare the output fields. In this case, an sentence
      @Override
      public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("sentence"));
      }
    }

Dedique un momento para leer los comentarios del código y entender cómo funciona este spout.

> [AZURE.NOTE] Aunque esta topología solo usa un spout, otras pueden tener varios que alimenten datos desde orígenes distintos en la topología.

###Creación de los bolts

Los bolts controlan el procesamiento de datos. Para esta topología, hay dos bolts:

* **SplitSentence**: divide las frases que emite **RandomSentenceSpout** en palabras individuales.

* **WordCount**: cuenta cuántas veces se ha repetido cada palabra.

> [AZURE.NOTE] Los bolts pueden hacer prácticamente todo; por ejemplo, cálculo, persistencia o hablar con componentes externos.

Cree dos archivos nuevos, **SplitSentence.java** y **WordCount.Java** en el directorio **src\\main\\java\\com\\microsoft\\example**. Use lo siguiente como contenido para los archivos:

**SplitSentence**

    package com.microsoft.example;

    import java.text.BreakIterator;

    import backtype.storm.topology.BasicOutputCollector;
    import backtype.storm.topology.OutputFieldsDeclarer;
    import backtype.storm.topology.base.BaseBasicBolt;
    import backtype.storm.tuple.Fields;
    import backtype.storm.tuple.Tuple;
    import backtype.storm.tuple.Values;

    //There are a variety of bolt types. In this case, we use BaseBasicBolt
    public class SplitSentence extends BaseBasicBolt {

      //Execute is called to process tuples
      @Override
      public void execute(Tuple tuple, BasicOutputCollector collector) {
        //Get the sentence content from the tuple
        String sentence = tuple.getString(0);
        //An iterator to get each word
        BreakIterator boundary=BreakIterator.getWordInstance();
        //Give the iterator the sentence
        boundary.setText(sentence);
        //Find the beginning first word
        int start=boundary.first();
        //Iterate over each word and emit it to the output stream
        for (int end=boundary.next(); end != BreakIterator.DONE; start=end, end=boundary.next()) {
          //get the word
          String word=sentence.substring(start,end);
          //If a word is whitespace characters, replace it with empty
          word=word.replaceAll("\\s+","");
          //if it's an actual word, emit it
          if (!word.equals("")) {
            collector.emit(new Values(word));
          }
        }
      }

      //Declare that emitted tuples will contain a word field
      @Override
      public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("word"));
      }
    }

**WordCount**

    package com.microsoft.example;

    import java.util.HashMap;
    import java.util.Map;
    import java.util.Iterator;

    import backtype.storm.Constants;
    import backtype.storm.topology.BasicOutputCollector;
    import backtype.storm.topology.OutputFieldsDeclarer;
    import backtype.storm.topology.base.BaseBasicBolt;
    import backtype.storm.tuple.Fields;
    import backtype.storm.tuple.Tuple;
    import backtype.storm.tuple.Values;
    import backtype.storm.Config;

    // For logging
    import org.apache.logging.log4j.Logger;
    import org.apache.logging.log4j.LogManager;

    //There are a variety of bolt types. In this case, we use BaseBasicBolt
    public class WordCount extends BaseBasicBolt {
        //Create logger for this class
        private static final Logger logger = LogManager.getLogger(WordCount.class);
        //For holding words and counts
        Map<String, Integer> counts = new HashMap<String, Integer>();
        //How often we emit a count of words
        private Integer emitFrequency;

        // Default constructor
        public WordCount() {
            emitFrequency=5; // Default to 60 seconds
        }

        // Constructor that sets emit frequency
        public WordCount(Integer frequency) {
            emitFrequency=frequency;
        }

        //Configure frequency of tick tuples for this bolt
        //This delivers a 'tick' tuple on a specific interval,
        //which is used to trigger certain actions
        @Override
        public Map<String, Object> getComponentConfiguration() {
            Config conf = new Config();
            conf.put(Config.TOPOLOGY_TICK_TUPLE_FREQ_SECS, emitFrequency);
            return conf;
        }

        //execute is called to process tuples
        @Override
        public void execute(Tuple tuple, BasicOutputCollector collector) {
            //If it's a tick tuple, emit all words and counts
            if(tuple.getSourceComponent().equals(Constants.SYSTEM_COMPONENT_ID)
                    && tuple.getSourceStreamId().equals(Constants.SYSTEM_TICK_STREAM_ID)) {
                for(String word : counts.keySet()) {
                    Integer count = counts.get(word);
                    collector.emit(new Values(word, count));
                    logger.info("Emitting a count of " + count + " for word " + word);
                }
            } else {
                //Get the word contents from the tuple
                String word = tuple.getString(0);
                //Have we counted any already?
                Integer count = counts.get(word);
                if (count == null)
                    count = 0;
                //Increment the count and store it
                count++;
                counts.put(word, count);
            }
        }

        //Declare that we will emit a tuple containing two fields; word and count
        @Override
        public void declareOutputFields(OutputFieldsDeclarer declarer) {
            declarer.declare(new Fields("word", "count"));
        }
    }

Dedique un momento a leer los comentarios del código para comprender cómo funciona cada bolt.

###Definición de la topología

La topología vincula los spouts y los bolts juntos en un gráfico, que define cómo fluyen los datos entre los componentes. También proporciona sugerencias de paralelismos que utiliza Storm al crear instancias de los componentes dentro del clúster.

Lo siguiente es un diagrama básico del gráfico de componentes para esta topología.

![diagrama que muestra la disposición de los spouts y bolts](./media/hdinsight-storm-develop-java-topology/wordcount-topology.png)

Para implementar la topología, cree un nuevo archivo denominado **WordCountTopology.java** en el directorio **src\\main\\java\\com\\microsoft\\example**. Use lo siguiente como contenido del archivo:

	package com.microsoft.example;

    import backtype.storm.Config;
    import backtype.storm.LocalCluster;
    import backtype.storm.StormSubmitter;
    import backtype.storm.topology.TopologyBuilder;
    import backtype.storm.tuple.Fields;

    import com.microsoft.example.RandomSentenceSpout;

    public class WordCountTopology {

      //Entry point for the topology
      public static void main(String[] args) throws Exception {
      //Used to build the topology
        TopologyBuilder builder = new TopologyBuilder();
        //Add the spout, with a name of 'spout'
        //and parallelism hint of 5 executors
        builder.setSpout("spout", new RandomSentenceSpout(), 5);
        //Add the SplitSentence bolt, with a name of 'split'
        //and parallelism hint of 8 executors
        //shufflegrouping subscribes to the spout, and equally distributes
        //tuples (sentences) across instances of the SplitSentence bolt
        builder.setBolt("split", new SplitSentence(), 8).shuffleGrouping("spout");
        //Add the counter, with a name of 'count'
        //and parallelism hint of 12 executors
        //fieldsgrouping subscribes to the split bolt, and
        //ensures that the same word is sent to the same instance (group by field 'word')
        builder.setBolt("count", new WordCount(), 12).fieldsGrouping("split", new Fields("word"));

        //new configuration
        Config conf = new Config();
        //Set to false to disable debug information
        // when running in production mode.
        conf.setDebug(false);

        //If there are arguments, we are running on a cluster
        if (args != null && args.length > 0) {
          //parallelism hint to set the number of workers
          conf.setNumWorkers(3);
          //submit the topology
          StormSubmitter.submitTopology(args[0], conf, builder.createTopology());
        }
        //Otherwise, we are running locally
        else {
          //Cap the maximum number of executors that can be spawned
          //for a component to 3
          conf.setMaxTaskParallelism(3);
          //LocalCluster is used to run locally
          LocalCluster cluster = new LocalCluster();
          //submit the topology
          cluster.submitTopology("word-count", conf, builder.createTopology());
          //sleep
          Thread.sleep(10000);
          //shut down the cluster
          cluster.shutdown();
        }
      }
    }

Dedique un momento a leer los comentarios del código para saber cómo se define la topología y cómo se envía después al clúster.

###registro

Storm utiliza Apache Log4j para registrar información. Si no configura el registro, la topología generará una gran cantidad de información de diagnóstico, que puede ser difícil de leer. Para controlar lo que se registra, cree un archivo denominado "__log4j2.xml__" en el directorio __resources__. Use lo siguiente como contenido del archivo:

    <?xml version="1.0" encoding="UTF-8"?>
    <Configuration>
    <Appenders>
        <Console name="STDOUT" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Logger name="com.microsoft.example" level="trace" additivity="false">
            <AppenderRef ref="STDOUT"/>
        </Logger>
        <Root level="error">
            <Appender-Ref ref="STDOUT"/>
        </Root>
    </Loggers>
    </Configuration>

Esta acción configura un nuevo registrador para la clase __com.microsoft.example__, que incluye los componentes de esta topología de ejemplo. El nivel está establecido para realizar un seguimiento de este registrador, con lo que se volverá a obtener cualquier información de registro que generen los componentes de esta topología. Si vuelve a observar el código de este proyecto, verá que solo el archivo WordCount.java implementa el registro; registrará el número de caracteres de cada palabra.

La sección `<Root level="error">` configura el nivel de registro raíz (todo lo que no esté en __com.microsoft.example__) para registrar solo la información de los errores.

> [AZURE.IMPORTANT] Aunque se reduce en gran medida la información registrada cuando se prueba una topología en el entorno de desarrollo, no se eliminan todos los datos de depuración generados al ejecutarse en un clúster de dicho entorno. Para reducir esa información, también debe establecer el valor de depuración en False en la configuración que se envía al clúster. Consulte el código de WordCountTopology.java en este documento para ver un ejemplo.

Para obtener más información sobre la configuración de registro de Log4j, consulte [http://logging.apache.org/log4j/2.x/manual/configuration.html](http://logging.apache.org/log4j/2.x/manual/configuration.html).

> [AZURE.NOTE] La versión de Storm 0.10.0 utiliza Log4j 2.x. Las versiones anteriores de Storm usaban Log4j 1.x, que empleaba otro formato de configuración del registro. Para obtener información sobre la configuración anterior, consulte [http://wiki.apache.org/logging-log4j/Log4jXmlFormat](http://wiki.apache.org/logging-log4j/Log4jXmlFormat).

##Prueba de la topología de manera local

Después de guardar los archivos, use el comando siguiente para probar localmente la topología.

	mvn compile exec:java -Dstorm.topology=com.microsoft.example.WordCountTopology

Mientras se ejecuta, la topología mostrará información de inicio. A continuación, comenzará a mostrar líneas similares a las siguientes, a medida que se emiten frases desde el spout y los bolts las procesan.

    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word snow
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word white
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 112 for word seven
    17:33:27 [Thread-16-count] INFO  com.microsoft.example.WordCount - Emitting a count of 195 for word the
    17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 113 for word and
    17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word dwarfs
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word snow

Si se consulta el registro del bolt WordCount, podemos comprobar que "and" se emitió 113 veces. El recuento continuará aumentando mientras se ejecute la topología porque el spout emite continuamente las mismas frases.

También habrá un intervalo de 5 segundos entre la emisión de las palabras y los recuentos. Esto se produce porque el componente __WordCount__ está configurado para emitir la información solo cuando llegue una tupla de marca, y solicita que esas tuplas se entreguen solo cada 5 segundos de forma predeterminada.

## Conversión de la topología a Flux

Flux es un nuevo marco de trabajo disponible con Storm 0.10.0, que permite separar la configuración de la implementación. Los componentes (bolts y spouts) se siguen definiendo en Java, pero la topología se define con un archivo YAML.

El archivo YAML define los componentes que se usarán para la topología, cómo circulan los datos entre ellos y qué valores se deben usar al inicializar los componentes. Puede incluir un archivo YAML como parte del archivo jar que contiene el proyecto cuando se implementa o puede usar un archivo YAML externo al iniciar la topología.

1. Saque el archivo __WordCountTopology.java__ del proyecto. Antes, este archivo definía la topología, pero no se usará con Flux.

2. En el directorio __resources__, cree un nuevo archivo llamado __topology.yaml__. Use el siguiente contenido para este archivo.

        # topology definition

        # name to be used when submitting. This is what shows up...
        # in the Storm UI/storm command-line tool as the topology name
        # when submitted to Storm
        name: "wordcount"

        # Topology configuration
        config:
        # Hint for the number of workers to create
        topology.workers: 1

        # Spout definitions
        spouts:
        - id: "sentence-spout"
            className: "com.microsoft.example.RandomSentenceSpout"
            # parallelism hint
            parallelism: 1

        # Bolt definitions
        bolts:
        - id: "splitter-bolt"
            className: "com.microsoft.example.SplitSentence"
            parallelism: 1

        - id: "counter-bolt"
            className: "com.microsoft.example.WordCount"
            constructorArgs:
            - 10
            parallelism: 1

        # Stream definitions
        streams:
        - name: "Spout --> Splitter" # name isn't used (placeholder for logging, UI, etc.)
            # The stream emitter
            from: "sentence-spout"
            # The stream consumer
            to: "splitter-bolt"
            # Grouping type
            grouping:
            type: SHUFFLE

        - name: "Splitter -> Counter"
            from: "splitter-bolt"
            to: "counter-bolt"
            grouping:
            type: FIELDS
            # field(s) to group on
            args: ["word"]

    Dedique un momento a leer a fondo y a entender lo que hace cada sección y cómo se relaciona con la definición basada en Java del archivo __WordCountTopology.java__.

3. Realice los siguientes cambios en el archivo __pom.xml__.

    * Agregue la siguiente dependencia nueva en la sección `<dependencies>`:

            <!-- Add a dependency on the Flux framework -->
            <dependency>
                <groupId>org.apache.storm</groupId>
                <artifactId>flux-core</artifactId>
                <version>${storm.version}</version>
            </dependency>

    * Agregue el complemento siguiente a la sección `<plugins>`. Este complemento administra la creación de un paquete (archivo jar) para el proyecto y aplica algunas transformaciones específicas a Flux cuando se crea el paquete.

            <!-- build an uber jar -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.3</version>
                <configuration>
                    <transformers>
                        <!-- Keep us from getting a "can't overwrite file error" -->
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer" />
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                        <!-- We're using Flux, so refer to it as main -->
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                            <mainClass>org.apache.storm.flux.Flux</mainClass>
                        </transformer>
                    </transformers>
                    <!-- Keep us from getting a bad signature error -->
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

    * En la sección __exec-maven-plugin__ `<configuration>`, cambie el valor de `<mainClass>` por `org.apache.storm.flux.Flux`. Esto permite a Flux controlar la ejecución de la topología cuando se ejecuta localmente en el entorno de desarrollo.

    * En la sección `<resources>`, agregue lo siguiente a `<includes>`. Esto incluye el archivo YAML que define la topología como parte del proyecto.
    
            <include>topology.yaml</include>

## Prueba local de la topología

1. Use lo siguiente para compilar y ejecutar la topología de Flux con Maven.

        mvn compile exec:java -Dexec.args="--local -R /topology.yaml"
    
    Si usa PowerShell, use lo siguiente:
    
        mvn compile exec:java "-Dexec.args=--local -R /topology.yaml"

    Si está en un sistema Linux, Unix u OS X y tiene [instalado Storm en el entorno de desarrollo](http://storm.apache.org/releases/0.10.0/Setting-up-development-environment.html), puede usar en su lugar los siguientes comandos:

        mvn compile package
        storm jar target/WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --local -R /topology.yaml

    El parámetro `--local` ejecuta la topología en modo local en el entorno de desarrollo. El parámetro `-R /topology.yaml` usa el recurso de archivo `topology.yaml` del archivo jar para definir la topología.

    Mientras se ejecuta, la topología mostrará información de inicio. A continuación, comenzará a mostrar líneas similares a las siguientes, a medida que se emiten frases desde el spout y los bolts las procesan.

        17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word snow
        17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word white
        17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 112 for word seven
        17:33:27 [Thread-16-count] INFO  com.microsoft.example.WordCount - Emitting a count of 195 for word the
        17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 113 for word and
        17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word dwarfs
    
    Habrá un retraso de 10 segundos entre los lotes de información registrada, porque el archivo `topology.yaml` pasa un valor de `10` cuando se crea el componente WordCount. Esto establece el intervalo de retraso de la tupla de marca en 10 segundos.

2.  Realice una copia del archivo `topology.yaml` del proyecto. Asígnele un nombre similar a `newtopology.yaml`. En el archivo, busque la siguiente sección y cambie el valor de `10` por `5`. Esto cambia el intervalo entre la emisión de lotes de recuentos de palabras de 10 segundos a 5.

          - id: "counter-bolt"
            className: "com.microsoft.example.WordCount"
            constructorArgs:
            - 5
            parallelism: 1

3. Para ejecutar la topología, use el siguiente comando:

        mvn exec:java -Dexec.args="--local /path/to/newtopology.yaml"

    O bien, si tiene Storm en su entorno de desarrollo de Unix, Linux u OS X:

        storm jar target/WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --local /path/to/newtopology.yaml

    Cambie `/path/to/newtopology.yaml` por la ruta de acceso al archivo newtopology.yaml que creó en el paso anterior. Este comando usará newtopology.yaml como definición de la topología. Puesto que no se incluyó el parámetro `compile`, Maven reutilizará la versión del proyecto compilada en los pasos anteriores.

    Una vez iniciada la topología, observará que el tiempo entre los lotes emitidos ha cambiado y ahora refleja el valor de newtopology.yaml. Esto demuestra que puede cambiar la configuración a través de un archivo YAML sin tener que volver a compilar la topología.

Hay otras características de Flux que no se tratan aquí, como la sustitución de variables en el archivo YAML en función de los parámetros pasados en tiempo de ejecución, o en variables de entorno. Para más información sobre estas y otras características del marco de trabajo Flux, consulte [Flux (https://storm.apache.org/releases/0.10.0/flux.html)](https://storm.apache.org/releases/0.10.0/flux.html).

##Trident

Trident es una abstracción de alto nivel que ofrece Storm. Admite el procesamiento con estado. La principal ventaja de Trident es que puede garantizar que todos los mensajes que entran en la topología se procesan una sola vez. Esto es difícil de lograr en una topología de Java, cuya garantía es que los mensajes se procesarán al menos una vez. También hay otras diferencias, como los componentes integrados que se pueden usar en lugar de crear bolts. De hecho, los bolts se reemplazan completamente por componentes menos genéricos, como filtros, proyecciones y funciones.

Las aplicaciones de Trident se pueden crear mediante proyectos de Maven. Siga los mismos pasos básicos que se mostraron anteriormente en este artículo; lo único diferente es el código. Actualmente, tampoco puede usarse Trident con el marco de trabajo Flux.

Para obtener más información sobre Trident, consulte <a href="http://storm.apache.org/documentation/Trident-API-Overview.html" target="_blank">Información general sobre la API de Trident</a>.

Para ver un ejemplo de una aplicación de Trident, consulte [Tendencias de Twitter con Apache Storm en HDInsight](hdinsight-storm-twitter-trending.md).

##Pasos siguientes

Ha aprendido a crear una topología de Storm con Java. Ahora obtenga información sobre:

* [Implementación y administración de topologías de Apache Storm en HDInsight](hdinsight-storm-deploy-monitor-topology.md)

* [Desarrollo de topologías de C# para Apache Storm en HDInsight con Visual Studio](hdinsight-storm-develop-csharp-visual-studio-topology.md)

Puede encontrar más topologías de ejemplo de Storm en [Topologías de ejemplo para Storm en HDInsight](hdinsight-storm-example-topology.md).

<!---HONumber=AcomDC_0921_2016-->