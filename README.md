Projet IDS ( Suricata - Wazuh )

dans host : 
sudo sysctl -w vm.max_map_count=262144

https://github.com/wazuh/wazuh-docker/tree/4.14.12/single-node

https://github.com/OISF/suricata/tree/main

docker compose exec curl bash -c "curl -sSL https://raw.githubusercontent.com/3CORESec/testmynids.org/master/tmNIDS -o /tmp/tmNIDS && chmod +x /tmp/tmNIDS && /tmp/tmNIDS"

docker exec -it $(docker ps -qf "name=wazuh.manager") grep -E "Cryptowall|Tor Relay" /var/ossec/logs/alerts/alerts.json

threat intelligence -> threat hunting -> Events
