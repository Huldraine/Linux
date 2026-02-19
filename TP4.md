# 🔐 TP – Administration SSH et Serveur Web Nginx

---

## 📑 Sommaire

- [A. Environnement virtualisé](#a-environnement-virtualisé)
- [B. Installation et configuration SSH](#b-installation-et-configuration-ssh)
- [C. Sécurisation du serveur SSH](#c-sécurisation-du-serveur-ssh)
- [D. Transfert de fichiers](#d-transfert-de-fichiers)
- [E. Analyse des logs et Fail2Ban](#e-analyse-des-logs-et-fail2ban)
- [F. Tunnels SSH](#f-tunnels-ssh)
- [G. Installation et configuration Nginx](#g-installation-et-configuration-nginx)
- [H. HTTPS et certificat auto-signé](#h-https-et-certificat-auto-signé)
- [I. Firewall et permissions](#i-firewall-et-permissions)
- [J. Validation finale](#j-validation-finale)

---

# A. Environnement virtualisé

## Configuration de la VM

- Ubuntu sous VirtualBox  
- 2 Go RAM minimum  
- 20 Go disque  
- Mode réseau : Bridged Adapter  

Vérification IP :

```bash
ip a
ping <IP_VM>
```

!(imageTP4/img1.png)

# B. Installation et configuration SSH

## 1) Installation du serveur SSH

```bash
sudo apt install openssh-server
```
2) Vérification du service SSH
sudo systemctl status ssh
sudo ss -tlnp | grep ssh

Connexion depuis la machine cliente :

ssh benoit@192.168.0.29

!(imageTP4/img2.png)

La connexion au serveur SSH est fonctionnelle.

3) Mise en place de l’authentification par clé
Génération de la clé sur la machine cliente
ssh-keygen

!(imageTP4/img3.png)

Copie de la clé publique vers le serveur
ssh-copy-id benoit@192.168.0.29

!(imageTP4/img4.png)

Connexion sans mot de passe validée :

!(imageTP4/img5.png)

L’authentification par clé est opérationnelle.

C. Sécurisation du serveur SSH

Modification du fichier de configuration :

sudo nano /etc/ssh/sshd_config

Paramètres modifiés :

PermitRootLogin no
PasswordAuthentication no
Port 2223

Redémarrage du service :

sudo systemctl restart ssh

Vérification du port personnalisé :

!(imageTP4/img6.png)

Test de connexion avec le nouveau port :

!(imageTP4/img7.png)

Le serveur SSH fonctionne maintenant sur le port 2223 avec authentification par clé uniquement.

Création d’un alias SSH

Édition du fichier client :

nano ~/.ssh/config

Contenu :

Host serveur-tp
    HostName 192.168.0.29
    User benoit
    Port 2223

!(imageTP4/img8.png)

Connexion simplifiée :

ssh serveur-tp

!(imageTP4/img9.png)

D. Transfert de fichiers
1) SCP
scp test.txt serveur-tp:/home/benoit/

!(imageTP4/img10.png)

Le fichier est correctement transféré vers le serveur.

2) SFTP
sftp serveur-tp
put fichier.txt
get fichier.txt
ls

Les commandes permettent de naviguer et transférer des fichiers.

3) RSYNC
rsync -avz dossier/ serveur-tp:/home/benoit/dossier/

Options utilisées :

-a : archive

-v : verbose

-z : compression

La synchronisation entre client et serveur fonctionne correctement.

E. Analyse des logs et Fail2Ban
1) Analyse des logs SSH
sudo tail -f /var/log/auth.log

!(imageTP4/img11.png)

On observe les connexions SSH et l’authentification par clé :

Accepted publickey

2) Installation de Fail2Ban
sudo apt install fail2ban

Configuration du fichier :

sudo nano /etc/fail2ban/jail.local

Contenu :

[sshd]
enabled = true
port = 2223
maxretry = 3
findtime = 60
bantime = 60

Redémarrage :

sudo systemctl restart fail2ban

Test de bannissement :

!(imageTP4/img12.png)

Fail2Ban bloque automatiquement les tentatives répétées.

F. Tunnels SSH
1) Tunnel local
ssh -L 8080:localhost:80 serveur-tp

Accès au service web distant via :

http://localhost:8080

!(imageTP4/img13.png)

2) Tunnel distant
ssh -R 9090:localhost:22 serveur-tp

Vérification :

curl http://localhost:9090

!(imageTP4/img14.png)

Le tunnel distant permet l’accès SSH via le serveur.

G. Installation et configuration Nginx
1) Installation
sudo apt install nginx

!(imageTP4/img15.png)

2) Création du site
sudo mkdir -p /var/www/site-tp

Création du fichier index.html :

<h1>HTTPS OK - site-tp</h1>

!(imageTP4/img16.png)

3) Configuration Nginx

Fichier :

/etc/nginx/sites-available/site-tp

!(imageTP4/img17.png)

Activation et vérification :

sudo ln -s /etc/nginx/sites-available/site-tp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
H. HTTPS et certificat auto-signé
1) Génération du certificat SSL
sudo mkdir -p /etc/nginx/ssl
cd /etc/nginx/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout site-tp.key -out site-tp.crt

!(imageTP4/img18.png)

2) Test HTTPS
curl -k https://192.168.0.29

!(imageTP4/img19.png)

3) Redirection HTTP → HTTPS
curl -I http://192.168.0.29

!(imageTP4/img20.png)

Le code 301 Moved Permanently confirme la redirection automatique.

I. Firewall et permissions
1) Configuration du firewall
sudo ufw allow 'Nginx Full'
sudo ufw status

!(imageTP4/img21.png)

Ports autorisés :

2223 (SSH)

80 (HTTP)

443 (HTTPS)

2) Permissions du site web
sudo chown -R www-data:www-data /var/www/site-tp
sudo chmod -R 755 /var/www/site-tp

Les permissions permettent à Nginx de lire les fichiers du site.

J. Validation finale
Élément	Statut
SSH sur port personnalisé	✅
Authentification par clé uniquement	✅
Root désactivé	✅
Fail2Ban actif	✅
SCP / SFTP / RSYNC	✅
Tunnel local & distant	✅
Nginx HTTP	✅
HTTPS fonctionnel	✅
Redirection HTTP → HTTPS	✅
Certificat auto-signé valide	✅
Firewall configuré	✅
Permissions correctes	✅
