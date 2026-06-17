docker run -d \
--name elasticsearch \
--net elastic-net \
-p 9200:9200 \
-e "discovery.type=single-node" \
-e "XPACK_SECURITY_ENABLED=false" \
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