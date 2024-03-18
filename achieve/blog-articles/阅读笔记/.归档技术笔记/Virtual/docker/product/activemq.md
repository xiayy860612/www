# Docker --- ActiveMQ

docker run --name='sds_prod_activemq' -d -v /data/activemq:/data \
-v /var/log/activemq:/var/log/activemq \
-p 8161:8161 \
-p 61616:61616 \
-p 61613:61613 \
webcenter/activemq:5.14.3