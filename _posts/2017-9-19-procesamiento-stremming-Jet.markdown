---
layout: post
title:  "Procesamiento Stremming con Hazelcast Jet"
date:   2017-5-2 22:06:44 +0100
categories: Python procesamiento streaming
---
## Introducción a Jet

Hazelcast Jet es un motor de procesamiento creado por Hazelcast para el procesamiento de grandes volumenes de datos en streaming.

Jet puede trabajar embebido en Hazelcast IMDG o por separado, pero aumentas sus prestaciones cuando lo hace junto IMDG.

La versión actual de Jet es la 0.4.

# Motor de procesamiento 

El motor de procesamiento se basa en el procesamiento de grafos acíclicos dirigidos. Básicamente el grafo modela el flujo de trabajo a través de un DAG (tipo de dato DAG). Cada vértice de este grafo es un paso en el flujo de trabajo y puede procesar, emitir y recibir datos.

Aunque los vértices se instancian a partir de la clase **Vertex**, podemos distinguir tres tipos de vértices distintos a efectos prácticos; los vértices **source** que no reciben la entrada de otro vértice pero envían a otros un flujo de información, los vertices **sink** desde los que se realiza la transferencia de información entre unos vértices y otros, y los vértices **internal** que reciben un flujo de información y producen uno de salida.

La forma en que se comunican los vértices es a través de los Edges (aristas).

Cuando nos referimos a una instancia de Jet hablamos de una unidad de procesamiento. Cada instancia se convierte en una unidad del clúster, desde la que se puede acceder a cualquier otra instancia del clúster.



## Ventajas y desventajas de Jet

Como ventaja principal podemos destacar su facilidad para ser programado, aunque esto puede complicarse por culpa de las implementaciones que debemos hacer de la case abstracta AbstractProcess para poder enviar procesos para que se ejecuten en un vértice.  También es útil cuando se apoya en IMDG para crear y mantener clústeres.

Por el contrario, Jet áun esta muy verde, actualmente se siguen implementando funcionalidades para suplir algunos problemas que son bastante graves y que sin duda suponen que se aplique menos a problemas reales, como por ejemplo:

  - Jet solo puede detectar un eror en el miembro del cluster que se ejecuta, y lo puede parar, pero el resto de vertices no es informado de el error, esta falta de tolerancia a fallos es muy grave y afecta mucho a las decisiones previas a la hora de implantar una tecnología u otra.
  - No admite la incorporación en caliente, un nuevo nodo no se unirá al cluster si este esta ejecutando un DAG, a no ser que se realice un replanteamiento de la tarea.



## Ejemplo de uso 

{% highlight Java %}
    public class ConsumeKafka {
      private static final int MESSAGE_COUNT_PER_TOPIC = 1_000_000;
      private EmbeddedZookeeper zkServer;
      private ZkUtils zkUtils;
      private KafkaServer kafkaServer;
      // Function main
      public static void main(String[] args) throws Exception {
          System.setProperty("hazelcast.logging.type", "log4j");
          new ConsumeKafka().run();
      }
      // Función principal
      private void run() throws Exception {
          createKafkaCluster();
          Runtime.getRuntime().addShutdownHook(new Thread(this::shutdownKafkaCluster));
          fillTopics();
          // Crear configuración para Jet 
          JetConfig cfg = new JetConfig();
          cfg.setInstanceConfig(new InstanceConfig().setCooperativeThreadCount(
                  Math.max(1, getRuntime().availableProcessors() / 2)));
           // Crear instancia con la configuración creada.
          JetInstance instance = Jet.newJetInstance(cfg);
          Jet.newJetInstance(cfg);
          // Declaramos IStreamMap para almacenar los datos
          IStreamMap<String, Integer> sinkMap = instance.getMap("sink");
          // Creamos un Job, para esto en primer lugar creamos un DAG
          DAG dag = new DAG();
          // Creamos las properties para el vértice source, que extraerá datos de KAfka
          Properties props = props(
                  "group.id", "group-" + Math.random(),
                  "bootstrap.servers", "localhost:9092",
                  "key.deserializer", StringDeserializer.class.getCanonicalName(),
                  "value.deserializer", IntegerDeserializer.class.getCanonicalName(),
                  "auto.offset.reset", "earliest");
          // Creamos el vertice "source" con las propierdades y los tópicos que va a leer.
          Vertex source = dag.newVertex("source", KafkaProcessors.streamKafka(props, "t1", "t2"));
          // Creamos un vértice que nos servirá para sincronizar y también para almacenar los datos en Hazelcast
          Vertex sink = dag.newVertex("sink", Sinks.writeMap("sink"));
          // Creamos la arista (Edge)
          dag.edge(between(source, sink));
          // Por ultimo creamos un nuevo Job.
          // Lo único que tenemos que hacer para crear un Job es pasar un DAG a una instancia.
          Job job = instance.newJob(dag);
          long start = System.nanoTime();
          job.execute();
          // Con este bucle podemos ver como vamos recibiendo los datos desde kafka, una vez se hayan guardado en Hazelcast.
          while (true) {
              int mapSize = sinkMap.size();
              System.out.format("Received %d entries in %d milliseconds.%n",
                      mapSize, NANOSECONDS.toMillis(System.nanoTime() - start));
              if (mapSize == MESSAGE_COUNT_PER_TOPIC * 2) {
                  break;
              }
              Thread.sleep(100);
          }
          shutdownKafkaCluster();
          System.exit(0);
      }
      // Creamos un Zookeeper y un Kafka  server
      private void createKafkaCluster() throws IOException {
          zkServer = new EmbeddedZookeeper();
          String zkConnect = "localhost:" + zkServer.port();
          ZkClient zkClient = new ZkClient(zkConnect, 30000, 30000, ZKStringSerializer$.MODULE$);
          zkUtils = ZkUtils.apply(zkClient, false);
          KafkaConfig config = new KafkaConfig(props(
                  "zookeeper.connect", zkConnect,
                  "broker.id", "0",
                  "log.dirs", Files.createTempDirectory("kafka-").toAbsolutePath().toString(),
                  "listeners", "PLAINTEXT://localhost:9092"));
          Time mock = new MockTime();
          kafkaServer = TestUtils.createServer(config, mock);
      }
      // Creamos dos tópicos y les añadimos información
      private void fillTopics() {
          createTopic(zkUtils, "t1", 32, 1, new Properties(), RackAwareMode.Disabled$.MODULE$);
          createTopic(zkUtils, "t2", 64, 1, new Properties(), RackAwareMode.Disabled$.MODULE$);
          System.out.println("Filling Topics");
          Properties props = props(
                  "bootstrap.servers", "localhost:9092",
                  "key.serializer", StringSerializer.class.getName(),
                  "value.serializer", IntegerSerializer.class.getName());
          try (KafkaProducer<String, Integer> producer = new KafkaProducer<>(props)) {
              for (int i = 1; i <= MESSAGE_COUNT_PER_TOPIC; i++) {
                  producer.send(new ProducerRecord<>("t1", "t1-" + i, i));
                  producer.send(new ProducerRecord<>("t2", "t2-" + i, i));
              }
              System.out.println("Published " + MESSAGE_COUNT_PER_TOPIC + " messages to t1");
              System.out.println("Published " + MESSAGE_COUNT_PER_TOPIC + " messages to t2");
          }
      }
      // Shudown kafka and zookeeper
      private void shutdownKafkaCluster() {
          kafkaServer.shutdown();
          zkUtils.close();
          zkServer.shutdown();
      }
      // Load properties
      private static Properties props(String... kvs) {
          final Properties props = new Properties();
          for (int i = 0; i < kvs.length;) {
              props.setProperty(kvs[i++], kvs[i++]);
          }
          return props;
      }
    }

{% endhighlight %}



## Referencias:

[https://github.com/hazelcast/hazelcast-jet/tree/master/hazelcast-jet-kafka](https://github.com/hazelcast/hazelcast-jet/tree/master/hazelcast-jet-kafka)

[https://hazelcast.com/products/jet/](https://hazelcast.com/products/jet/)


