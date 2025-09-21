# SOC-open-source
guide clair et structuré pour installer et configurer Wazuh dans un contexte d’entreprise ou de lab. Je le présente en 4 grandes phases : préparation, installation du serveur (Wazuh Manager + Dashboard), déploiement des agents, puis intégration à la stack ELK (incluse dans le package officiel).


Voici un **guide clair et structuré** pour installer et configurer **Wazuh** (dernière version stable) dans un contexte d’entreprise ou de lab.
Je le présente en 4 grandes phases : préparation, installation du serveur (Wazuh Manager + Dashboard), déploiement des agents, puis intégration à la stack ELK (incluse dans le package officiel).

---

## 1️⃣  Prérequis

| Élément                  | Recommandations                                                                           |
| ------------------------ | ----------------------------------------------------------------------------------------- |
| **Système**              | Linux 64 bits (Ubuntu 20.04/22.04 ou CentOS/RHEL 8/9).                                    |
| **Ressources minimales** | 4 vCPU, 8 Go RAM, 50 Go disque (plus selon volume de logs).                               |
| **Accès**                | Compte sudo/ root, ports 443 (dashboard), 1514/udp et 1515/tcp (agents), 55000/tcp (API). |
| **Mises à jour**         | `apt update && apt upgrade` (ou `yum update`) avant de commencer.                         |

---

## 2️⃣  Installation « tout-en-un » (Wazuh Manager + OpenSearch + Dashboard)

La méthode officielle (script) installe :

* **Wazuh Manager** (moteur SIEM),
* **OpenSearch** (moteur de recherche/logs),
* **Wazuh Dashboard** (interface Web).

### Étapes

1. **Télécharger le script officiel**

   ```bash
   curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
   ```

2. **Rendre exécutable et lancer**

   ```bash
   chmod +x wazuh-install.sh
   sudo ./wazuh-install.sh -a
   ```

   L’option `-a` = installation « all-in-one ».

3. **À la fin** le script affiche :

   * l’URL du **dashboard** (https\://\<votre\_IP>),
   * l’utilisateur/mot de passe initial.

> ⚠️ Vérifie que le pare-feu autorise les ports : 443, 1514/udp, 1515/tcp, 55000/tcp.

---

## 3️⃣  Déploiement des Agents

Installe l’agent sur chaque machine à superviser (Linux, Windows ou macOS).

Exemple **Ubuntu/Debian** :

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
echo "deb https://packages.wazuh.com/4.x/apt stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update
sudo apt install wazuh-agent
```

Configurer le fichier `/var/ossec/etc/ossec.conf` :

```xml
<server>
  <address>IP_du_Manager</address>
  <port>1514</port>
</server>
```

Puis activer et démarrer :

```bash
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

Pour **Windows** : télécharge l’agent MSI depuis le Dashboard (section Agents).

---

## 4️⃣  Vérifications et intégrations

* **Dashboard** : se connecter à `https://IP_Manager` avec les identifiants fournis.
* Vérifier dans l’onglet **Agents** que les machines apparaissent en « Active ».
* Configurer les **règles**, **alertes**, et les intégrations (email, Slack, etc.).

---

## 5️⃣  Bonnes pratiques post-installation

1. **SSL/TLS** : générer des certificats valides (Let’s Encrypt ou CA interne) pour le dashboard et l’API.
2. **Sauvegarde** : planifier backup régulier d’OpenSearch (snapshots).
3. **Alerting** : personnaliser les règles et seuils pour éviter le bruit.
4. **Supervision** : surveiller les ressources (CPU/RAM) et ajuster la rétention des logs.

---

### Documentation utile

* Guide officiel : [https://documentation.wazuh.com](https://documentation.wazuh.com)
* Référence pour intégration avec Elastic/ELK, AWS, etc.

---

En suivant ces étapes, tu obtiens un **SOC open-source opérationnel** :
le manager central collecte et corrèle les événements, les agents remontent les logs, et le dashboard offre la visualisation et la configuration en temps réel.
