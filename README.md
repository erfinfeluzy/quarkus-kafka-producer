# Tutorial Quarkus.io : Produce Kafka Topic

Tutorial ini akan melakukan hands on mengenai cara publish message ke kafka topic dengan menggunakan Quarkus


Prerequsite tutorial ini adalah:
- Java 8 
- Maven **3.6.2 keatas** -> versi dibawah ini tidak didukung
- IDE kesayangan anda (RedHat CodeReady, Eclipse, VSCode, Netbeans, InteliJ, dll)
- Git client
- Dasar pemrograman Java
- Untuk build native image diperlukan Docker runtime


```
### Konfigurasi Kafka Producer
```properties
mp.messaging.outgoing.mytopic-publisher.connector=smallrye-kafka
mp.messaging.outgoing.mytopic-publisher.topic=mytopic
mp.messaging.outgoing.mytopic-publisher.value.serializer=org.apache.kafka.common.serialization.StringSerializer
```

### Publish Random message ke Kafka Server setiap 5 detik

Snippet berikut pada file *KafkaTopicGenerator.java* untuk mempublish random data ke Topic dengan nama **mytopic**.
```java
	@Inject
	@Channel("mytopic-publisher")
	private Emitter<String> emitter;

	@Scheduled(every = "5s")
	public void scheduler() {
		
		//randomly generate kafka message to topic:mytopic every 5 seconds	
		emitter.send( "Data tanggal : " + new Date () + "; id : " + UUID.randomUUID() );
	}
```
> Note:
> 1. **@Channel("mytopic-publisher")** adalah channel stream internal microprofile. *mytopic-pulisher* dikonfigurasikan pada application.properties untuk diteruskan ke Kafka Server dengan nama topic : *mytopic*.
> 2. **@Scheduled(every = "5s")** digunakan untuk mensimulasikan traffic data masuk ke Kafka Server setiap 5 detik.


## Bonus! Deploy aplikasi kamu ke Red Hat Openshift

### Step 1: Build aplikasi sebagai native container
```bash
$ mvn clean package -Dnative -Dquarkus.native.container-build=true
```
Perintah ini akan generate aplikasi yang secara native run as container. setelah langkah ini cek file ** \*runner\* ** di foler target
```bash
$ ls -altr target/
...
-rw-r--r--  1 erfinfeluzy  staff      4509 Jun 15 08:15 quarkus-kafka-producer-1.0-SNAPSHOT.jar
drwxr-xr-x  3 erfinfeluzy  staff        96 Jun 15 08:15 generated-sources
drwxr-xr-x  3 erfinfeluzy  staff        96 Jun 15 08:15 maven-archiver
drwxr-xr-x  3 erfinfeluzy  staff        96 Jun 15 08:15 maven-status
drwxr-xr-x  5 erfinfeluzy  staff       160 Jun 15 08:15 classes
-rwxr-xr-x  1 erfinfeluzy  staff  41647448 Jun 15 08:18 quarkus-kafka-producer-1.0-SNAPSHOT-runner
drwxr-xr-x  4 erfinfeluzy  staff       128 Jun 15 08:18 quarkus-kafka-producer-1.0-SNAPSHOT-native-image-source-jar
drwxr-xr-x  9 erfinfeluzy  staff       288 Jun 15 08:27 .
drwxr-xr-x  9 erfinfeluzy  staff       288 Jun 15 10:46 ..
```

### Step 2: Build aplikasi sebagai native container
```bash
$ docker build -f src/main/docker/Dockerfile.native -t quarkus/kafka-producer:v3 .
```
```
### Step 3: Deploy image ke Registry
kali ini saya menggunakan **Quay.io** sebagai registry, karena memiliki fitur untuk security scan image kita.
PS: saya menggunakan skopeo untuk mempermudah perpindahan registry
```bash
$ skopeo --insecure-policy copy --dest-creds=$CREDENTIAL docker-daemon:quarkus/kafka-producer:v3 docker://quay.io/efeluzy/quarkus-kafka-producer:v3
```
untuk credential pada Quay.io dapat di atur di konfigurasi security pada Quay.io

### Step 4: Deploy image ke Openshift
kali saya akan menggunakan Openshift CLI untuk mendeploy aplikasi. 
> PS: developer dapat menggunakan web console untuk cara yang lebih mudah
```bash
$ oc new-app quay.io/efeluzy/quarkus-kafka-producer:v3 --name quarkus-kafka-producer 
```
