## 制作镜像
由于认证证书实际上就是相同的证书拷贝至其他节点，不如直接先生成证书，然后打包新镜像使用。

- Dockerfile

```bash
FROM elasticsearch:7.6.2
USER root
#生成证书
RUN bin/elasticsearch-certutil ca --out config/elastic-stack-ca.p12 --pass NBEZfka8
RUN bin/elasticsearch-certutil cert --ca config/elastic-stack-ca.p12 --ca-pass NBEZfka8 --out config/elastic-certificates.p12 --pass NBEZfka8
#创建keystore
RUN bin/elasticsearch-keystore create
#将密码添加至keystore
RUN sh -c '/bin/echo -e "NBEZfka8" | sh bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password'
RUN sh -c '/bin/echo -e "NBEZfka8" | sh bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password'
RUN chmod 777 /usr/share/elasticsearch/config/elastic-certificates.p12
RUN chmod 777 /usr/share/elasticsearch/config/elastic-stack-ca.p12

```
- 创建镜像

docker build -t ycspace-elasticsearch:7.6.2 .

## 准备配置文件
### elasticsearch.yml
```bash
network.host: 0.0.0.0
#master节点es01
cluster.initial_master_nodes: ["es01"]
discovery.seed_hosts: ["es01","es02","es03"]
cluster.name: "es-docker-cluster"
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: Authorization,X-Requested-With,Content-Length,Content-Type
#开启kibana监控配置,如果不开启，也可以在kibana监控界面开启
xpack.monitoring.collection.enabled: true
#开启安全认证相关配置
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.audit.enabled: true
xpack.license.self_generated.type: basic
xpack.security.transport.ssl.keystore.type: PKCS12
xpack.security.transport.ssl.verification_mode: certificate
#名字要和自定义镜像中的名字一致
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.type: PKCS12
```
### kibana.yml
```bash
server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://192.168.0.74:9200", "http://192.168.0.74:9201" , "http://192.168.0.74:9202" ]
elasticsearch.username: kibana
elasticsearch.password: uGdHoolqbswIEoc9vssD
i18n.locale: "zh-CN"
```
### docker-compose.yml
```bash
version: '2.2'
services:
  es01:
    image: ycspace-elasticsearch:7.6.2
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01
      - bootstrap.memory_lock=true
      - "TAKE_FILE_OWNERSHIP=true"
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
      - TZ=Asia/Shanghai
      - node.master=true
      - node.data=true
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - http.cors.allow-headers=Authorization,X-Requested-With,Content-Length,Content-Type
      - xpack.security.enabled=true
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.audit.enabled=true
      - xpack.license.self_generated.type=basic
      - xpack.monitoring.collection.enabled=true
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /data/service/soft/es01/data:/usr/share/elasticsearch/data
      - /data/service/soft/es01/logs:/usr/share/elasticsearch/logs
      - ./plugins:/usr/share/elasticsearch/plugins
      - ./elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    ports:
      - 9200:9200
    networks:
      - elastic

  es02:
    image: ycspace-elasticsearch:7.6.2
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01
      - bootstrap.memory_lock=true
      - "TAKE_FILE_OWNERSHIP=true"
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
      - TZ=Asia/Shanghai
      - node.master=true
      - node.data=true
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - http.cors.allow-headers=Authorization,X-Requested-With,Content-Length,Content-Type
      - xpack.security.enabled=true
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.audit.enabled=true
      - xpack.license.self_generated.type=basic
      - xpack.monitoring.collection.enabled=true
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /data/service/soft/es02/data:/usr/share/elasticsearch/data
      - /data/service/soft/es02/logs:/usr/share/elasticsearch/logs
      - ./plugins:/usr/share/elasticsearch/plugins
      - ./elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    ports:
      - 9201:9200
    networks:
      - elastic

  es03:
    image: ycspace-elasticsearch:7.6.2
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01
      - bootstrap.memory_lock=true
      - "TAKE_FILE_OWNERSHIP=true"
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
      - TZ=Asia/Shanghai
      - node.master=true
      - node.data=true
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - http.cors.allow-headers=Authorization,X-Requested-With,Content-Length,Content-Type
      - xpack.security.enabled=true
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.audit.enabled=true
      - xpack.license.self_generated.type=basic
      - xpack.monitoring.collection.enabled=true
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /data/service/soft/es03/data:/usr/share/elasticsearch/data
      - /data/service/soft/es03/logs:/usr/share/elasticsearch/logs
      - ./plugins:/usr/share/elasticsearch/plugins
      - ./elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    ports:
      - 9202:9200
    networks:
      - elastic

  kibana:
    depends_on:
      - es01
      - es02
      - es03
    image: kibana:7.6.2
    container_name: kibana
    ports:
      - 5601:5601
    environment:
      - elasticsearch.url=http://es01:9200
      - elasticsearch.hosts=http://es01:9200
      - i18n.locale=zh-CN   
      - TZ=Asia/Shanghai
    volumes:
      - ./kibana.yml:/usr/share/kibana/config/kibana.yml
      - /etc/localtime:/etc/localtime
    networks:
      - elastic

networks:
  elastic:
    driver: bridge
```
## 启动容器
### 创建数据目录
```bash
mkdir /data/service/soft/es01/{logs,data} -p
mkdir /data/service/soft/es02/{logs,data} -p
mkdir /data/service/soft/es03/{logs,data} -p
```
### 启动
```bash
docker-compose up -d
```

## 配置用户名密码
- 进入master节点执行 `./bin/elasticsearch-setup-passwords interactive --verbose` 根据提示设置密码即可，注意密码设置要与事先准备好的`kibana.yml`文件里的一致
- 设置好后可以进入kibana进行验证，也可以通过curl查看节点状态

```bash
[root@dev-02 ~]# curl --user elastic:uGdHoolqbswIEoc9vssD http://127.0.0.1:9200/_cat/nodes?v
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
172.26.0.4           38          97   7    2.74    2.22     1.55 dilm      -      es03
172.26.0.3           22          97   7    2.74    2.22     1.55 dilm      -      es02
172.26.0.2           23          97   7    2.74    2.22     1.55 dilm      *      es01

```

## 问题
### 1.权限问题
默认es没有权限写入本地挂载的数据目录，可以通过在docker-compose文件添加` "TAKE_FILE_OWNERSHIP=true"`参数，或者把数据目录权限改成777 `chmod -R 777 /data/service/soft/es0*`
### 2.docker-compose文件变量不生效问题
docker-compose中 kibana配置 变量好像没有生效，最后通过挂载kibana.yml才正常运行，暂时没弄懂
