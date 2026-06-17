docker run -d \
--name elasticsearch \
--net elastic-net \
-p 9200:9200 \
-e "discovery.type=single-node" \
-e "xpack.security.enabled=false" \
-e "xpack.security.http.ssl.enabled=false" \
-e "xpack.security.transport.ssl.enabled=false" \
-e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
-v es-data:/usr/share/elasticsearch/data \
docker.elastic.co/elasticsearch/elasticsearch:8.12.0

http://localhost:9200


docker run -d \
--name kibana \
--net elastic-net \
-p 5601:5601 \
-e "ELASTICSEARCH_HOSTS=http://elasticsearch:9200" \
docker.elastic.co/kibana/kibana:8.12.0

等待 1-2 分钟，浏览器访问 http://localhost:5601

# 分词器
https://github.com/infinilabs/analysis-ik
You can download the packaged plugins from here:
https://release.infinilabs.com/


# 1. 清理旧容器
docker rm -f kibana elasticsearch

# 2. 清理旧数据卷（必须清，否则旧的配置会干扰）
docker volume rm es-data

# 3. 启动带 IK 分词器的 ES
docker run -d \
--name elasticsearch \
--net elastic-net \
-p 9200:9200 \
-e "discovery.type=single-node" \
-e "xpack.security.enabled=false" \
-e "xpack.security.http.ssl.enabled=false" \
-e "xpack.security.transport.ssl.enabled=false" \
-e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
-v es-data:/usr/share/elasticsearch/data \
es-ik:8.12.0

# 4. 等待 30 秒让 ES 彻底启动，然后启动 Kibana
sleep 30 && docker run -d \
--name kibana \
--net elastic-net \
-p 5601:5601 \
-e "ELASTICSEARCH_HOSTS=http://elasticsearch:9200" \
docker.elastic.co/kibana/kibana:8.12.0