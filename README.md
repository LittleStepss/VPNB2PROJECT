# VPNB2PROJECT

L'erreur "No such file or directory" se produit parce que le script `build-ca` n'est pas directement accessible après l'installation d'OpenVPN et easy-rsa. Nous devons d'abord configurer easy-rsa correctement. Voici les étapes à suivre pour configurer easy-rsa et corriger cette erreur.

### Configuration d'easy-rsa

1. **Installation d'OpenVPN et easy-rsa** :
    ```bash
    sudo apt update
    sudo apt install openvpn easy-rsa
    ```

2. **Copie des fichiers easy-rsa dans le répertoire de travail** :
    ```bash
    make-cadir ~/openvpn-ca
    cd ~/openvpn-ca
    ```

3. **Initialisation du répertoire easy-rsa** :
    ```bash
    cp -r /usr/share/easy-rsa/* .
    ```

4. **Modification du fichier `vars` pour ajuster les paramètres** :
    Ouvrir le fichier `vars` avec un éditeur de texte :
    ```bash
    nano vars
    ```

    Et modifier les lignes suivantes en fonction de vos besoins :
    ```bash
    set_var EASYRSA_REQ_COUNTRY    "FR"
    set_var EASYRSA_REQ_PROVINCE   "YourProvince"
    set_var EASYRSA_REQ_CITY       "YourCity"
    set_var EASYRSA_REQ_ORG        "YourOrganization"
    set_var EASYRSA_REQ_EMAIL      "YourEmail@example.com"
    set_var EASYRSA_REQ_OU         "YourOrganizationalUnit"
    ```

5. **Sourcer le fichier `vars`** :
    ```bash
    source vars
    ```

6. **Nettoyer le répertoire de certificats précédents (le cas échéant)** :
    ```bash
    ./easyrsa clean-all
    ```

7. **Construire l'Autorité de Certification (CA)** :
    ```bash
    ./easyrsa build-ca
    ```

### Création des certificats et des clés

1. **Générer la clé et le certificat pour le serveur** :
    ```bash
    ./easyrsa build-server-full server nopass
    ```

2. **Générer la clé Diffie-Hellman** :
    ```bash
    ./easyrsa gen-dh
    ```

3. **Générer le fichier de clé TLS auth** :
    ```bash
    openvpn --genkey --secret ta.key
    ```

4. **Générer les clés et certificats pour les clients** (par exemple, pour `client1`) :
    ```bash
    ./easyrsa build-client-full client1 nopass
    ```

### Configuration du Serveur OpenVPN

1. **Créer un fichier de configuration `/etc/openvpn/server.conf`** :
    ```bash
    sudo nano /etc/openvpn/server.conf
    ```

    Et y ajouter les lignes suivantes :
    ```conf
    port 1194
    proto udp
    dev tun
    ca /etc/openvpn/ca.crt
    cert /etc/openvpn/server.crt
    key /etc/openvpn/server.key
    dh /etc/openvpn/dh.pem
    server 10.8.0.0 255.255.255.0
    ifconfig-pool-persist /var/log/openvpn/ipp.txt
    push "redirect-gateway def1 bypass-dhcp"
    push "dhcp-option DNS 8.8.8.8"
    push "dhcp-option DNS 8.8.4.4"
    keepalive 10 120
    tls-auth /etc/openvpn/ta.key 0
    cipher AES-256-CBC
    user nobody
    group nogroup
    persist-key
    persist-tun
    status /var/log/openvpn/openvpn-status.log
    log-append /var/log/openvpn/openvpn.log
    verb 3
    ```

2. **Copier les fichiers générés dans le répertoire d'OpenVPN** :
    ```bash
    sudo cp ~/openvpn-ca/ta.key /etc/openvpn/
    sudo cp ~/openvpn-ca/pki/ca.crt /etc/openvpn/
    sudo cp ~/openvpn-ca/pki/private/server.key /etc/openvpn/
    sudo cp ~/openvpn-ca/pki/issued/server.crt /etc/openvpn/
    sudo cp ~/openvpn-ca/pki/dh.pem /etc/openvpn/
    ```

3. **Configurer le firewall** :
    ```bash
    sudo ufw allow 1194/udp
    sudo ufw enable
    ```

4. **Démarrer et activer le service OpenVPN** :
    ```bash
    sudo systemctl start openvpn@server
    sudo systemctl enable openvpn@server
    ```

### Configuration du Client VPN

1. **Copier les fichiers nécessaires vers le client** :
    - `ca.crt`, `client1.crt`, `client1.key`, `ta.key`.

2. **Créer un fichier de configuration client** :
    - Pour un client Linux, créer `/etc/openvpn/client.conf` :
    ```conf
    client
    dev tun
    proto udp
    remote YOUR_SERVER_IP 1194
    resolv-retry infinite
    nobind
    persist-key
    persist-tun
    ca ca.crt
    cert client1.crt
    key client1.key
    remote-cert-tls server
    tls-auth ta.key 1
    cipher AES-256-CBC
    verb 3
    ```

3. **Connecter le client au VPN** :
    ```bash
    sudo openvpn --config /etc/openvpn/client.conf
    ```
