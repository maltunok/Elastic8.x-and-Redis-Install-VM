
# MASTER NODE
sudo wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

sudo apt-get install apt-transport-https

echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

sudo apt-get update && sudo apt-get install elasticsearch
#save the password

sudo systemctl daemon-reload

sudo systemctl start elasticsearch.service

sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
#save the token

sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
#save the token

sudo nano /etc/elasticsearch/elasticsearch.yml

...
cluster.name: es-cluster
node.name: elastic30
node.roles: [master, data , ingest, remote_cluster_client]
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: false
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["elastic30", "elastic31"]
xpack.security.enabled: false
xpack.security.enrollment.enabled: true
xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
http.host: 0.0.0.0
...

sudo systemctl restart elasticsearch.service
sudo systemctl status elasticsearch.service

#INSTALL KIBANA TO MASTER NODE
sudo apt-get update && sudo apt-get install kibana
sudo nano /etc/kibana/kibana.yml
...
server.port: 5601
server.host: 0.0.0.0
server.publicBaseUrl: "http://<IP of VM>/"
server.name: "elastic30"
elasticsearch.hosts: ["http://10.0.0.4:9200/", "http://10.0.0.5:9200/"]
elasticsearch.serviceAccountToken:<Kibana Token>
logging:
  appenders:
    file:
      type: file
      fileName: /var/log/kibana/kibana.log
      layout:
        type: json
  root:
    appenders:
      - default
      - file
pid.file: /run/kibana/kibana.pid
...

sudo systemctl restart kibana.service
sudo systemctl status kibana.service

# DATA NODE
sudo wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install elasticsearch
sudo /usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <node token>
sudo nano /etc/elasticsearch/elasticsearch.yml
...
cluster.name: es-cluster
node.name: elastic31
node.roles: [master, data, ingest, remote_cluster_client]
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: false
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["elastic30", "elastic31"]
xpack.security.enabled: false
xpack.security.enrollment.enabled: true
xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
discovery.seed_hosts: ["10.0.0.4:9300", "10.0.0.5"]
http.host: 0.0.0.0
...

sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
sudo systemctl status elasticsearch.service
