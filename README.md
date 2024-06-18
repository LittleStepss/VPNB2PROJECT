# VPNB2PROJECT

Pour copier les fichiers nécessaires depuis le serveur VPN vers le client, vous pouvez utiliser des outils comme `scp` (secure copy) pour transférer les fichiers de manière sécurisée. Voici les étapes détaillées pour effectuer cette opération :

### 1. Localisation des Fichiers sur le Serveur VPN

Assurez-vous que les fichiers suivants sont bien situés dans le répertoire correct sur votre serveur VPN :
- `ca.crt` : `/etc/openvpn/ca.crt`
- `client1.crt` : `/etc/openvpn/client1.crt`
- `client1.key` : `/etc/openvpn/client1.key`
- `ta.key` : `/etc/openvpn/ta.key`

### 2. Transfert des Fichiers vers le Client

#### Utilisation de `scp`

Supposons que l'utilisateur du client s'appelle `clientuser` et que l'adresse IP ou le nom de domaine du client soit `client_ip_or_domain`.

Sur le serveur VPN, exécutez les commandes suivantes pour transférer les fichiers :

```bash
# Transfert du fichier ca.crt
scp /etc/openvpn/ca.crt clientuser@client_ip_or_domain:/home/clientuser/

# Transfert du fichier client1.crt
scp /etc/openvpn/client1.crt clientuser@client_ip_or_domain:/home/clientuser/

# Transfert du fichier client1.key
scp /etc/openvpn/client1.key clientuser@client_ip_or_domain:/home/clientuser/

# Transfert du fichier ta.key
scp /etc/openvpn/ta.key clientuser@client_ip_or_domain:/home/clientuser/
```

#### Utilisation de `rsync` (option alternative)

Vous pouvez également utiliser `rsync` pour transférer les fichiers, ce qui est particulièrement utile pour synchroniser des fichiers entre deux machines :

```bash
# Transfert du fichier ca.crt
rsync -avz /etc/openvpn/ca.crt clientuser@client_ip_or_domain:/home/clientuser/

# Transfert du fichier client1.crt
rsync -avz /etc/openvpn/client1.crt clientuser@client_ip_or_domain:/home/clientuser/

# Transfert du fichier client1.key
rsync -avz /etc/openvpn/client1.key clientuser@client_ip_or_domain:/home/clientuser/

# Transfert du fichier ta.key
rsync -avz /etc/openvpn/ta.key clientuser@client_ip_or_domain:/home/clientuser/
```

### 3. Configuration du Client VPN

Sur le client, vous devez déplacer ces fichiers vers un répertoire sécurisé et créer le fichier de configuration OpenVPN.

1. **Déplacer les fichiers dans le répertoire OpenVPN** :

```bash
sudo mv /home/clientuser/ca.crt /etc/openvpn/
sudo mv /home/clientuser/client1.crt /etc/openvpn/
sudo mv /home/clientuser/client1.key /etc/openvpn/
sudo mv /home/clientuser/ta.key /etc/openvpn/
```

2. **Créer le fichier de configuration du client** :

Créez un fichier `/etc/openvpn/client.conf` et ajoutez-y les configurations suivantes :

```conf
client
dev tun
proto udp
remote YOUR_SERVER_IP 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca /etc/openvpn/ca.crt
cert /etc/openvpn/client1.crt
key /etc/openvpn/client1.key
remote-cert-tls server
tls-auth /etc/openvpn/ta.key 1
cipher AES-256-CBC
verb 3
```

Remplacez `YOUR_SERVER_IP` par l'adresse IP publique de votre serveur VPN.

3. **Lancer la connexion VPN** :

Pour démarrer la connexion VPN sur le client, exécutez :

```bash
sudo openvpn --config /etc/openvpn/client.conf
```

### Résumé

En suivant ces étapes, vous avez sécurisé les fichiers nécessaires depuis le serveur VPN vers le client, configuré le client avec les certificats et les clés nécessaires, et établi une connexion VPN. Vous êtes maintenant prêt à utiliser votre VPN pour accéder à l'intranet depuis l'extérieur.
