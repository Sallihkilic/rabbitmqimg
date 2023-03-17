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

![alt text](https://github.com/Sallihkilic/rabbitmqimg/blob/master/images/image-asset.png?raw=true)

<p> <u>Classic mirrored queues</u> partition boyunca bölünmüş olanlar, partitionun her iki tarafında birer lider olacak ve yine her iki tarafta bağımsız hareket edecek. </p>

<p><code>pause_minority</code> gibi bir <u>partition işleme stratejisi </u>kullanılmak üzere yapılandırılmadığı sürece network connectivity geri yüklendikten sonra bile bölünme devam edecektir.</p>
<br>
<br>
<p>Optimum performans elde etmek için kuyruklarınızın her zaman olabildiğince kısa olduğundan emin olun. Daha uzun kuyruklar daha fazla işlem yükü getirir. Optimum performans için sıraların her zaman 0 civarında kalmasını öneririz.</p>

![alt text](https://github.com/Sallihkilic/rabbitmqimg/blob/master/images/federated-queue.gif?raw=true)

![alt text](https://github.com/Sallihkilic/rabbitmqimg/blob/master/images/federated-queue.gif?raw=true)

<h3 style=color:#902550;>
<strong>3.  SUSPEND VE RESUME'UN NEDEN OLDUĞU PARTITIONLAR</strong><h3>



















<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>



<p style=font-size:20px;><u>Yukarıda anlatılan konuyu özetlemek gerekirse;</u></p>

<li><b> En Son Kararlı Sürümü Kullanın </b> <br>
Clusterınızda RabbitMQ'nun en son kararlı sürümünü kullanmanız önerilir. En son sürüm, hata düzeltmeleri, güvenlik yamaları ve performans iyileştirmeleri içerir. Sisteminizdeki diğer bileşenlerle uyumlu kalmasını sağlamak için RabbitMQ clusterınızı güncel tutmanız da önemlidir.</li>

<br>
<br>
<li><b> Clusterınızı High Availability için Tasarlayın </b> <br>
RabbitMQ clusterınızı tasarlarken High Availabilityi göz önünde bulundurmanız önemlidir. Yüksek düzeyde kullanılabilir bir cluster, bir veya daha fazla node un arızalanmasına direnebilir ve çalışmaya devam edebilir. High Availability elde etmek için, RabbitMQ'yu clusterdaki nodes arasında queues and exchanges leri çoğaltacak şekilde yapılandırabilirsiniz. Bu, bir node başarısız olursa, clusterdaki diğer nodeların mesajları işlemeye devam edebilmesini sağlar.
</li>

<br>
<br>

<li><b>Load balancer kullanın</b> <br>
Bir load balancer, gelen trafiği clusterdaki birden çok RabbitMQ node una  dağıtabilir. Bu, tek bir node un performans sorunlarına yol açabilecek trafiğe boğulmamasını sağlamaya yardımcı olur. Ek olarak, bir load balancer bir node un ne zaman başarısız olduğunu algılayabilir ve trafiği cluster da kalan nodelara yönlendirebilir.</li>

<br>
<br>

<li><b>Kaynak Sınırlarını Yapılandırın</b><br>
Çok fazla CPU, bellek veya disk alanı tüketmemelerini sağlamak için RabbitMQ nodelerinin kaynak sınırlarını yapılandırmak önemlidir. Kaynak limitleri belirleyerek, bir node un clusterdaki diğer nodeların performansını etkilemesini önleyebilirsiniz. Ek olarak, kaynak limitleri, nodeların çökmesine neden olabilecek out-of-memory hatalarını önlemeye yardımcı olabilir.</li>

<br>
<br>

<li><b>Clusterınızı İzleyin (Monitor)</b><br>
RabbitMQ clusterınızı izlemek, kararlı kalmasını ve iyi performans göstermesini sağlamak için çok önemlidir. CPU kullanımı, memory kullanımı, disk kullanımı ve mesaj çıkışı gibi ölçümleri izlemelisiniz. Bu ölçümleri görselleştirmek ve sorunlar olduğunda sizi uyarmak için Prometheus ve Grafana gibi izleme araçları kullanılabilir.</li>

<br>
<br>

<li><b>Backup Your Data</b><br>
RabbitMQ clusterınızdaki verileri düzenli olarak yedeklemeniz önemlidir. Bu, bir node arızalanırsa kayıp verileri kurtarabilmenizi sağlar. RabbitMQ, rabbitmqctl aracı gibi verileri yedeklemek ve geri yüklemek için araçlar sağlar.</li>

<br>
<br>

<li><b>Clusterınızı Test Edin</b><br>
RabbitMQ clusterınızı üretime dağıtmadan önce kapsamlı bir şekilde test etmeniz önemlidir. Test, üretim trafiğini etkilemeden önce tüm yapılandırma sorunlarının veya performans darboğazlarının belirlenmesine yardımcı olabilir. Clusterınızı yüksek trafik hacimleri ve node arızaları gibi çeşitli senaryolar altında test etmelisiniz.
</li>


<br><br><br>
<p>Bu best practiceleri izleyerek RabbitMQ clusterının sabit kalmasını ve iyi performans göstermesini sağlayabilirsiniz. Clusterı high availability için tasarlamanız, bir load balancer kullanmanız, kaynak limitlerini yapılandırmanız, clusterı izlemeniz, datayı yedeklemeniz ve clusterınızı üretime dağıtmadan önce test etmeniz önemlidir.