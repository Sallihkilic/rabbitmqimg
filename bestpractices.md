<h1><strong>RabbitMq  Best Practices <strong></h1><br>

![alt text](https://github.com/Sallihkilic/rabbitmqimg/blob/master/images/baslik.png?raw=true)
<br>

<h2 style=color:red;><storng>Clustering and Network Partitions</strong><h2>
<br>
<p style=font-size:15px;>Küme üyeleri arasındaki ağ bağlantısı hatalarının, veri tutarlılığı ve müşteri işlemleri için kullanılabilirlik (CAP teoreminde olduğu gibi) üzerinde etkisi vardır. Farklı uygulamaların tutarlılık konusunda farklı gereksinimleri olduğundan ve kullanılamamayı farklı bir ölçüde tolere edebildiğinden, farklı bölüm işleme stratejileri (bu stratejileri aşağıda açıklanacaktır bkz.) mevcuttur.</p>

<h3 style=color:#902550;>
<strong>1. AĞ BÖLÜMLERİNİ ALGILAMA</strong><h3>
<br>

<p style=font-size:15px;>Bir node başka bir node ile belli bir süre (defalut 60 sec) iletişim kuramazsa onun kapalı olup olmadğını belirler. Eğer iki düğüm tekrar iletişime geçerse ikisi de birbirinin kapalı olduğunu düşünür. Nodes partition olduğunu belirleyecektir. RabbitMQ logu aşağıdaki gibidir.</p>

    2020-05-18 06:55:37.324 [error] <0.341.0> Mnesia(rabbit@warp10): ** ERROR ** mnesia_event got {inconsistent_database, running_partitioned_network, rabbit@hostname2}


<p style=font-size:15px;> Partition'lar loglar, HTTP API veya CLI command lar ile belirleneblir.</p>

<p style=font-size:15px;> <i> <code>rabbitmq-diagnostics cluster_status</code> </i>normalde partitonlar için boş bir liste gösterir.

![alt text](https://github.com/Sallihkilic/rabbitmqimg/blob/master/images/2023-03-16%2007_21_33-Clustering%20and%20Network%20Partitions%20—%20RabbitMQ.png?raw=true)


<p style=font-size:15px;> Ancak Network Partition oluşmuşsa partiton lar ile ilgili bilgiler böyle gözükecektir.</p>


![alt text](https://github.com/Sallihkilic/rabbitmqimg/blob/master/images/2023-03-16%2007_45_47-Clustering%20and%20Network%20Partitions%20—%20RabbitMQ.png?raw=true)


<p style=font-size:15px;>HTTP API partition bilgisini <code>GET /api/nodes</code> endpointlerinde <code> partitionların </code>  altındaki her node için döndürür.

Bir partiton oluşmuşsa Management UI, overview sayfasında uyarıyı gösterecektir. 

 </p>

<h3 style=color:#902550;>
<strong>2. NETWORK PARTITION SIRASINDA DAVRANIŞ</strong><h3>
<br>

<p> Bir network partition sırasında, cluster iki veya daha fazla tarafta bağımsız olarak gelişebilir ve her iki tarafta diğerinin crashed olduğunu düşünür. Bu senaryo split-brain olarak bilinir. Queues,bindings,exchanges ayrı ayrı oluşturulabilir veya silinebilir.</p>


<p> <u>Quorum queues</u> majority tarafta  yeni bir lider seçecektir. Quorum queues minority tarafındaki replikalar artık ilerleme kaydetmeyecek (yeni mesaj kabul etme,tüketicilere iletme vb.) tüm bu işler yeni lider tarafından yapılacaktır. </p>

<p> <u>Classic mirrored queues</u> partition boyunca bölünmüş olanlar, partitionun her iki tarafında birer lider olacak ve yine her iki tarafta bağımsız hareket edecek. </p>

<p><code>pause_minority</code> gibi bir <u>partition işleme stratejisi </u>kullanılmak üzere yapılandırılmadığı sürece network connectivity geri yüklendikten sonra bile bölünme devam edecektir.</p>


<h3 style=color:#902550;>
<strong>3.  SUSPEND VE RESUME'UN NEDEN OLDUĞU PARTITIONLAR</strong><h3>

