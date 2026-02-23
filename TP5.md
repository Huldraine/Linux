Partie 1 – Prise en main et sécurisation

1. Accès à l’interface

Connectez-vous à l’interface web de pfSense.
Questions :

![img1](imageTP5/img1)


Quelle est l’adresse IP du LAN  ?
192.168.168.56.101

Quelle est l’adresse IP du WAN ?
10.0.2.15

Pourquoi utilise-t-on HTTPS ?
c'est sécurisé

Pourquoi faut-il changer les identifiants par défaut sur un pare-feu ?

car c'est un mdp set par défaut donc tout le monde à le même au début c'est plus sécurisé

2. Sécurisation de l’accès administrateur

![img1](imageTP5/img2)

Modifiez les paramètres du compte administrateur.
Questions :


Où se gèrent les utilisateurs ?
dans system>User manager

Qu’est-ce qu’un mot de passe robuste ?
un mdp robust est un mdp avec plus de 12 caractères, des caractères spéciaux, des majuscules ou minuscules, qui est unique et imprévisible.

Pourquoi sécuriser en priorité l’accès admin sur un équipement réseau ?
car il peut gérer les autres utilisateurs et à accès aux paramètres du réseau

Partie 2 – Comprendre les interfaces réseau

3. Vérification des interfaces

Vérifiez l’affectation WAN / LAN.

![img1](imageTP5/img2)

Questions :


Quelle interface permet l’accès Internet ?
WAn

Quelle interface correspond au réseau interne ?
LAN

Que se passerait-il si les interfaces étaient inversées ?

Cela peut entrainer des problèmes d'accès à l'interace web et de repérage des interfaces adsl et sdsl

Partie 3 – Configuration des services réseau

4. DHCP

Configurez le serveur DHCP pour le réseau LAN.

La gateway

![img1](imageTP5/img31)

Questions :


Pourquoi utiliser DHCP plutôt qu’une IP fixe ?
pour éviter d'avoir 2 appareils avec 2 même ip (conflit ip)

Quelle plage d’adresses choisir ?
192.168.56.50 --> 192.168.56.200

![img1]](image-TP5/img5.png)

Quelles adresses faut-il éviter d’inclure dans la plage ?
192.168.56.255, 192.168.56.0, 192.168.56.254

Vérification :


Ubuntu obtient-elle automatiquement une adresse IP ?

oui

![img1]](image-TP5/img4.png)

5. DNS

Activez et configurez le résolveur DNS.

![img1]](image-TP5/img8.png)
![img1]](image-TP5/img7.png)

Questions :


Pourquoi un pare-feu peut-il jouer le rôle de serveur DNS ?

- Centralisation des requêtes DNS
- Mise en cache des réponses
- Possibilité de filtrage
- Amélioration de la sécurité

Que se passe-t-il si le DNS ne fonctionne pas mais que le ping vers 8.8.8.8 fonctionne ?

Le routage est fonctionnel, mais la résolution des noms de domaine est défaillante.

Partie 4 – Autoriser l’accès Internet

6. Règles de pare-feu

Configurez les règles nécessaires pour permettre aux machines du LAN d’accéder à Internet.
info :
pfSense applique les règles de haut en bas.
Questions :


Quelle doit être la source ?
LAN net
Parce que le trafic part du réseau interne.

Quelle doit être la destination ?
any
Parce que les machines du LAN doivent accéder à Internet.

Faut-il autoriser tous les protocoles ?
En lab : oui (pour tester)

En production : non (on limiterait à :

TCP 80 (HTTP)

TCP 443 (HTTPS)

UDP 53 (DNS)

ICMP si besoin)

Tests :


Ping vers pfSense
Ping vers 8.8.8.8
Test DNS
Accès web

![img1]](image-TP5/img37.png)
![img1]](image-TP5/img10.png)
![img1]](image-TP5/img9.png)

7. NAT

Vérifiez la configuration du NAT sortant.
![img1]](image-TP5/img11.png)
![img1]](image-TP5/img12.png)

Questions :


Pourquoi le NAT est-il nécessaire avec une interface WAN en NAT ?

Parce que :

Les IP privées (192.168.x.x) ne sont pas routables sur Internet.

Le NAT permet de masquer plusieurs IP privées derrière une seule IP WAN.

Quelle est la différence entre NAT automatique et manuel ?
Automatic :

pfSense crée les règles automatiquement.

Manual :

L’administrateur définit précisément les règles.

Utilisé en environnement avancé.

Comment vérifier qu’une traduction d’adresse a lieu ?

Dans Diagnostics > States,
on observe que :

192.168.56.102 est traduite en 10.0.2.15 (adresse WAN).

Cela confirme que le NAT fonctionne correctement.

Partie 5 – Filtrage

8. Blocage d’un site spécifique

Bloquez l’accès à un site web de votre choix.

![img1]](image-TP5/img13.png)
![img1]](image-TP5/img14.png)


Questions :


## Réponses aux questions

### Faut-il bloquer par IP ou par nom de domaine ?

Il est préférable de bloquer par nom de domaine (FQDN).
Les sites utilisent plusieurs adresses IP (CDN) qui peuvent changer.
Le blocage par IP est donc moins fiable et plus difficile à maintenir.

---

### Que se passe-t-il si le site utilise HTTPS ?

Avec HTTPS, le contenu est chiffré.
Le pare-feu ne voit que l’adresse IP et le port 443.
Il ne peut pas filtrer des pages ou contenus spécifiques.

---

### Pourquoi le blocage par IP peut-il être contourné ?

Le blocage par IP peut être contourné car :
- Les sites possèdent plusieurs IP.
- Les IP peuvent changer.
- L’utilisateur peut utiliser un VPN.
- Le trafic peut passer en IPv6.

Testez et observez les logs.

![img1]](image-TP5/img15.png)
![img1]](image-TP5/img16.png)

9. Blocage d’une catégorie de sites (jeux d’argent)

Créez une solution propre et maintenable pour bloquer plusieurs sites.

![img1]](image-TP5/img17.png)
![img1]](image-TP5/img18.png)
![img1]](image-TP5/img19.png)
![img1]](image-TP5/img20.png)
tips : réfléchissez à l’intérêt des alias.

## Questions – Blocage d’une catégorie de sites

### Pourquoi ne pas créer une règle par site ?

Créer une règle par site rend la configuration complexe et difficile à maintenir.
L’utilisation d’un alias permet de regrouper plusieurs sites dans une seule règle,
ce qui simplifie la gestion et améliore la lisibilité.

---

### Où se créent les alias ?

Les alias se créent dans :
Firewall > Aliases.

---

### Comment vérifier qu’une règle bloque réellement le trafic ?

On vérifie dans :
Status > System Logs > Firewall.

Les logs doivent montrer :
- L’adresse IP source (machine du LAN)
- L’adresse IP de destination
- L’action "Block"

Partie 6 – Aller plus loin (partie plus tendue)

10. Blocage par catégorie (réseaux sociaux)


Créez un alias pour une nouvelle catégorie.

![img1]](image-TP5/img21.png)

Implémentez une règle.

![img1]](image-TP5/img22.png)

Analysez les logs.
![img1]](image-TP5/img23.png)
![img1]](image-TP5/img24.png)

Question :

Que se passe-t-il si la règle est placée sous une règle "Pass Any" ?

Si la règle de blocage est placée sous une règle "Pass Any",
elle ne sera jamais appliquée.
pfSense analyse les règles de haut en bas et applique la première correspondance.
La règle "Pass Any" autorisera donc le trafic avant que la règle de blocage ne soit évaluée.

11. Règles horaires


Créez un horaire.

![img1]](image-TP5/img25.png)

Appliquez-le à une règle existante.

![img1]](image-TP5/img26.png)

Questions :

Pourquoi les règles horaires sont-elles utiles en entreprise ?

Les règles horaires permettent d’activer ou désactiver certaines règles de pare-feu
en fonction de plages horaires définies.

En entreprise, elles sont utiles pour :

- Limiter l’accès à certains services (réseaux sociaux, streaming) pendant les heures de travail  
- Appliquer une politique de sécurité adaptée aux horaires d’activité  
- Réduire les usages non professionnels  
- Automatiser des restrictions sans intervention manuelle  

Elles permettent donc d’adapter le filtrage aux besoins réels de l’organisation.

12. Serveur web local

Installez un serveur web sur Ubuntu.

![img1]](image-TP5/img27.png)
![img1]](image-TP5/img28.png)

Objectifs :

Autoriser un accès spécifique
Bloquer les autres

![img1]](image-TP5/img29.png)

Questions :

🔹 Filtrer par IP source ?

Oui.

On filtre par IP source pour autoriser uniquement une machine précise à accéder au serveur web.
Cela permet de contrôler qui a le droit d’accéder au service.

🔹 Filtrer par port ?

Oui.

On filtre par port 80 (HTTP) car on souhaite protéger uniquement le service web, pas tout le trafic vers la machine.
Le filtrage par port permet de contrôler quel service est accessible.

🔹 Pourquoi le pare-feu protège-t-il le LAN même en réseau interne ?

Parce que tout le trafic entre les machines du réseau passe par l’interface LAN de pfSense (192.168.56.1).

pfSense filtre tous les paquets qui traversent une interface, même si le trafic est interne au réseau privé.

Donc le pare-feu contrôle aussi les communications à l’intérieur du LAN.

13. Logs et analyse

Activez la journalisation sur certaines règles.

![img1]](image-TP5/img30.png)

Questions :


🔹 Différence entre paquet bloqué et autorisé

Paquet autorisé (Pass) :
Le pare-feu laisse passer le trafic vers sa destination.

Paquet bloqué (Block) :
Le pare-feu refuse le trafic, la communication ne peut pas s’établir.

Dans les logs, l’action indiquée est Pass ou Block.

🔹 Identifier quelle règle a déclenché le blocage

Dans Status → System Logs → Firewall, chaque entrée indique :

L’interface (ex : LAN)

L’action (Block / Pass)

Les adresses IP source et destination

Le port utilisé

Le numéro ou l’identifiant de la règle

Le numéro de règle permet de retrouver exactement la règle responsable dans Firewall → Rules.

🔹 Comprendre le sens du trafic

Dans les logs :

Source = machine qui envoie le paquet

Destination = machine qui reçoit le paquet

Exemple :
192.168.56.11 → 192.168.56.10 : 80

Cela signifie que la machine 192.168.56.11 tente d’accéder au port 80 du serveur 192.168.56.10.

Le sens du trafic permet de comprendre qui initie la communication.

15. Filtrage MAC

Testez le filtrage par adresse MAC.

![img1]](image-TP5/img32.png)
![img1]](image-TP5/img33.png)

Question :


🔹 Le filtrage MAC est-il réellement sécurisé ?

Non.

Le filtrage par adresse MAC n’est pas réellement sécurisé.
Il peut limiter l’accès dans un environnement simple, mais il ne constitue pas une protection fiable.

🔹 Pourquoi est-il facilement contournable ?

Parce que :

Une adresse MAC peut être modifiée facilement (MAC spoofing).

Il suffit d’observer le réseau pour connaître une adresse MAC autorisée.

Un attaquant peut configurer sa carte réseau avec cette adresse.

Donc le filtrage MAC repose sur une information facilement falsifiable, contrairement à une authentification forte ou à un filtrage par IP sécurisé.

16. Portail captif

Implémentez un portail captif.

![img1]](image-TP5/img34.png)

Questions :

🔹 Dans quels contextes utilise-t-on cela ?

On utilise un portail captif dans :

Hôtels

Aéroports

Universités

Entreprises

Wi-Fi public

Il sert à contrôler l’accès des utilisateurs au réseau.

🔹 Quel(s) avantage(s) par rapport à une simple règle de pare-feu ?

Un portail captif permet :

D’authentifier les utilisateurs

D’identifier qui se connecte

D’appliquer des restrictions par utilisateur

De présenter des conditions d’utilisation

Une simple règle de pare-feu :

Autorise ou bloque une IP

Ne gère pas l’authentification utilisateur

Le portail captif offre donc un contrôle plus précis et traçable.

17. Sauvegarde / restauration


Sauvegardez la configuration.

![img1]](image-TP5/img35.png)

Modifiez-la.
Restaurez-la.

![img1]](image-TP5/img36.png)

Question :

🔹 Pourquoi la sauvegarde régulière est-elle essentielle en production ?

Parce qu’elle permet :

De restaurer rapidement le système en cas d’erreur de configuration

De récupérer après une panne ou une attaque

D’éviter une interruption prolongée du service réseau

Sans sauvegarde, une mauvaise configuration peut rendre le réseau inutilisable.

