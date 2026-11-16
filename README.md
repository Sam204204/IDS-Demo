# IDS-Demo : Détection d'Intrusions avec Wazuh & Suricata

Ce projet propose une démonstration d'une architecture de détection d'intrusions. Il associe **Suricata** (NIDS) pour l'analyse du trafic réseau et **Wazuh** (SIEM & HIDS) pour la centralisation, la corrélation et la visualisation des alertes de sécurité.

## Architecture du Projet

* **Wazuh (Single-Node v4.14.12)** : Collecte, analyse et indexe les journaux système (HIDS) ainsi que les alertes remontées par Suricata (SIEM).
* **Suricata (NIDS)** : Inspecte le trafic réseau en temps réel à l'aide de règles de détection et génère des alertes au format `eve.json`.
* **Conteneur `curl` (Simulateur d'attaques)** : Permet de générer du trafic malveillant simulé pour valider le bon fonctionnement des règles de détection.

### Structure du dépôt

```text
IDS-Demo/
├── config/              
├── curl/                 
├── suricata/             
├── docker-compose.yml    
└── README.md             
```
##  Prérequis
* **Configuration mémoire système** : L'indexeur Wazuh (OpenSearch) nécessite l'augmentation du paramètre `vm.max_map_count` sur l'hôte Linux.
Exécute la commande suivante sur la machine hôte avant le premier démarrage :
```bash
sudo sysctl -w vm.max_map_count=262144
```

## Déploiement
**Générer les certificats SSL pour l'indexeur Wazuh (obligatoire au premier démarrage) :**
Exécute la commande suivante dans config/wazuh/ :
   ```bash
   docker compose -f generate-indexer-certs.yml run --rm generator
   ```
**Construire et démarrer les conteneurs en arrière-plan :**
   ```bash
   docker compose up -d
   ```
**Nettoyer le déploiement précédent :**
   ```bash
   docker compose down -v
   ```
**Accéder à l'interface web Wazuh :**
   * Ouvre le navigateur à l'adresse `https://localhost`.
   * *Identifiants par défaut : `admin` / `SecretPassword`.*

## Règle de Détection Personnalisée (Suricata)

Une règle locale est intégrée dans `suricata/rules/local.rules` et fusionnée automatiquement au démarrage :

```suricata
alert http any any -> any any (msg:"DEMO - Injection SQL"; http.uri; content:"OR"; nocase; content:"1=1"; sid:1000001; rev:1;)
```

Cette signature détecte toute requête HTTP sortante ou entrante dont l'URL contient simultanément les chaînes « OR » et « 1=1 »

---

## Simulation d'Attaques et Tests 
### Scénario 1 : Déclencher la règle personnalisée (`Injection SQL`)
Envoie une requête HTTP contenant une injection SQL (`OR 1=1`) dans l'URL depuis le conteneur `curl` :

```bash
docker compose exec curl curl -g "http://example.com/?user=admin'+OR+1=1"
```
### Scénario 2 : Simuler des menaces connues avec `tmNIDS`
Pour vérifier que Suricata détecte bien les menaces réseau et transmet les alertes à Wazuh, nous utilisons le script **tmNIDS** (TestMyNIDS) depuis le conteneur `curl`.
Exécute la commande suivante pour simuler des activités malveillantes  :

```bash
docker compose exec curl bash -c "curl -sSL https://raw.githubusercontent.com/3CORESec/testmynids.org/master/tmNIDS -o /tmp/tmNIDS && chmod +x /tmp/tmNIDS && /tmp/tmNIDS"
```
## Vérification des Alertes
### 1. Via le terminal
Interroge directement le fichier `alerts.json` du conteneur `wazuh.manager` pour filtrer les signatures déclenchées :
```bash
docker exec -it $(docker ps -qf "name=wazuh.manager") grep -E "Cryptowall|Tor Relay" /var/ossec/logs/alerts/alerts.json
```
### 2. Via le Dashboard Wazuh
1. Se connecte à l'interface web Wazuh.
2. Ouvre le menu principal en haut à gauche.
3. Navigue vers **Threat Intelligence** ➔ **Threat Hunting** ➔ onglet **Events**.
4. Observe les alertes générées par Suricata en temps réel.
---
## Références et Ressources

* [Wazuh Docker (v4.14.12 Single-Node)](https://github.com/wazuh/wazuh-docker/tree/4.14.12/single-node)
* [Suricata GitHub Repository (OISF)](https://github.com/OISF/suricata/tree/main)
* [Framework TestMyNIDS (3CORESec)](https://github.com/3CORESec/testmynids.org)
* https://github.com/nbittich/grafana-example
