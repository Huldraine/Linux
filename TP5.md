# Partie 1 – Prise en main et sécurisation

## 1. Accès à l’interface

Connectez-vous à l’interface web de pfSense et observez l’écran suivant :

![img1](imageTP5/img1)

**Questions :**

- **Quelle est l’adresse IP du LAN ?**  
  L’interface LAN est configurée sur `192.168.168.56.101` (valeur affichée en haut de la page).

- **Quelle est l’adresse IP du WAN ?**  
  L’adresse WAN est `10.0.2.15`, elle représente l’interface qui fait face à l’Internet.

- **Pourquoi utilise‑t‑on HTTPS ?**  
  Parce que HTTPS chiffre la communication avec la console d’administration, empêchant l’écoute clandestine des identifiants et des données de configuration.

- **Pourquoi faut‑il changer les identifiants par défaut sur un pare‑feu ?**  
  Les comptes « admin » sont livrés avec un mot de passe standard connu de tous. Le modifier empêche qu’un attaquant qui connaît ces identifiants accède à l’appareil et mette en danger le réseau.

## 2. Sécurisation de l’accès administrateur

Sur l’onglet `System > User Manager` vous pouvez modifier le compte `admin` :

![img1](imageTP5/img2)

> *La capture montre l’interface de gestion des utilisateurs où l’on change le mot de passe et, si nécessaire, crée un nouveau compte administrateur.*

**Questions :**

- **Où se gèrent les utilisateurs ?**  
  Dans le menu **System > User Manager** de l’interface pfSense.

- **Qu’est‑ce qu’un mot de passe robuste ?**  
  Un mot de passe robuste fait au moins 12 caractères, mélange majuscules, minuscules, chiffres et symboles, et ne correspond à aucun mot courant ; il est unique à chaque compte.

- **Pourquoi sécuriser en priorité l’accès admin sur un équipement réseau ?**  
  L’administrateur peut modifier toutes les configurations. S’il est compromis, l’attaquant prend le contrôle du pare‑feu et peut faire sauter les protections, installer des backdoors ou espionner le trafic.

# Partie 2 – Comprendre les interfaces réseau

## 3. Vérification des interfaces

Vérifiez que les interfaces WAN et LAN sont correctement affectées. L’image ci‑dessous montre l’écran de statut des interfaces :

![img1](imageTP5/img2)

**Questions :**

- **Quelle interface permet l’accès Internet ?**  
  Le port `WAN` relie pfSense à l’Internet.

- **Quelle interface correspond au réseau interne ?**  
  L’interface `LAN` dessert le réseau local.

- **Que se passerait‑il si les interfaces étaient inversées ?**  
  Le réseau interne ne pourrait plus accéder à l’Internet ; les paquets sortants seraient rejetés et l’accès à l’interface d’administration pourrait devenir confus ou inaccessible, car le routage serait incorrect.

# Partie 3 – Configuration des services réseau

## 4. DHCP

Activez le serveur DHCP pour le réseau LAN et définissez une plage d’adresses :

![img1](imageTP5/img31)

**Questions :**

- **Pourquoi utiliser DHCP plutôt qu’une IP fixe ?**  
  Le DHCP automatise l’attribution des adresses, évite les conflits d’adresse et simplifie la gestion lorsque des machines se connectent ou se déconnectent fréquemment.

- **Quelle plage d’adresses choisir ?**  
  Une plage adaptée pourrait être `192.168.56.50` à `192.168.56.200` afin de laisser de l’espace pour les adresses statiques en dehors de ce segment.

> L’illustration ci‑dessus montre la configuration de la passerelle et la zone de DHCP.

![img1](image-TP5/img5.png)

- **Quelles adresses faut‑il éviter d’inclure dans la plage ?**  
  Les adresses réseau (`192.168.56.0`), de broadcast (`192.168.56.255`) et la passerelle (`192.168.56.254`) ne doivent pas se retrouver dans la plage.

**Vérification :**

Après activation, un client Ubuntu obtient automatiquement une adresse :

![img1](image-TP5/img4.png)

## 5. DNS

Activez le résolveur DNS intégré et configurez‑le selon les captures :

![img1](image-TP5/img8.png)
![img1](image-TP5/img7.png)

**Questions :**

- **Pourquoi un pare‑feu peut‑il jouer le rôle de serveur DNS ?**  
  Il centralise les requêtes des clients, met en cache les réponses pour accélérer les résolutions, permet un filtrage (bloquer des noms malveillants) et protège contre le spoofing.

- **Que se passe‑t‑il si le DNS ne fonctionne pas mais que le ping vers `8.8.8.8` fonctionne ?**  
  Cela signifie que le routage IP est opérationnel, mais que la résolution des noms de domaine est défaillante : les machines ne peuvent plus traduire les noms en adresses IP.

# Partie 4 – Autoriser l’accès Internet

## 6. Règles de pare‑feu

Créez une règle de sortie sur l’interface LAN afin que les machines internes puissent accéder à Internet. Rappel : pfSense évalue les règles de haut en bas.

**Questions :**

- **Quelle doit être la source ?**  
  `LAN net`, car le trafic part du réseau interne.

- **Quelle doit être la destination ?**  
  `any`, pour autoriser l’accès vers n’importe quel hôte externe.

- **Faut‑il autoriser tous les protocoles ?**  
  En laboratoire, on peut autoriser tous les protocoles pour tester ; en production on limiterait aux services nécessaires (TCP 80, 443, UDP 53, ICMP, etc.).

**Tests :**

Effectuez un ping vers pfSense, vers `8.8.8.8`, testez une résolution DNS et ouvrez un site Web :

![img1](image-TP5/img37.png)
![img1](image-TP5/img10.png)
![img1](image-TP5/img9.png)

## 7. NAT

Vérifiez la configuration du NAT sortant :

![img1](image-TP5/img11.png)
![img1](image-TP5/img12.png)

**Questions :**

- **Pourquoi le NAT est‑il nécessaire avec une interface WAN en NAT ?**  
  Les adresses privées (192.168.x.x) ne sont pas routables sur Internet. Le NAT transforme ces adresses en l’adresse publique du WAN (`10.0.2.15`), permettant à plusieurs hôtes de partager une seule IP.

- **Quelle est la différence entre NAT automatique et manuel ?**  
  En mode automatique, pfSense génère les règles de traduction pour chaque réseau; en mode manuel l’administrateur crée lui‑même les règles, pratique dans des scénarios complexes.

- **Comment vérifier qu’une traduction d’adresse a lieu ?**  
  Dans `Diagnostics > States`, on voit qu’une entrée montre par exemple `192.168.56.102` traduite en `10.0.2.15`. Cela confirme que les paquets sortent avec l’adresse WAN.

# Partie 5 – Filtrage

## 8. Blocage d’un site spécifique

Pour empêcher l’accès à un site, créez une règle de blocage et observez les captures :

![img1](image-TP5/img13.png)
![img1](image-TP5/img14.png)

**Questions :**

### Faut‑il bloquer par IP ou par nom de domaine ?  
Il est préférable de bloquer par nom de domaine (FQDN) car les sites utilisent souvent plusieurs adresses IP (CDN) qui peuvent changer dynamiquement. Un blocage par IP est fragile et nécessite une maintenance continue.

### Que se passe‑t‑il si le site utilise HTTPS ?  
Le trafic est chiffré ; seul l’IP de destination et le port (443) sont visibles par le pare‑feu. Il est impossible de filtrer des pages ou des URI à ce niveau, seul le blocage par nom ou adresse reste possible.

### Pourquoi le blocage par IP peut‑il être contourné ?  
Les utilisateurs peuvent passer par un VPN, le site peut changer ses adresses, ou le trafic IPv6 peut être employé. Les IPs multiples et variables évitent les filtrages statiques.

Vous pouvez consulter les journaux pour confirmer le blocage :

![img1](image-TP5/img15.png)
![img1](image-TP5/img16.png)

## 9. Blocage d’une catégorie de sites (jeux d’argent)

Regroupez plusieurs domaines dans un alias et appliquez‑le à une règle.

![img1](image-TP5/img17.png)
![img1](image-TP5/img18.png)
![img1](image-TP5/img19.png)
![img1](image-TP5/img20.png)

**Questions – Blocage d’une catégorie de sites**

- **Pourquoi ne pas créer une règle par site ?**  
  Une règle par site rend la configuration verbeuse et difficile à faire évoluer. Un alias permet de maintenir une seule règle regroupant tous les domaines de la catégorie.

- **Où se créent les alias ?**  
  Dans le menu **Firewall > Aliases**.

- **Comment vérifier qu’une règle bloque réellement le trafic ?**  
  En consultant `Status > System Logs > Firewall`. Les entrées indiquent la source, la destination, l’action « Block » et le numéro de règle correspondant.

# Partie 6 – Aller plus loin (partie plus tendue)

## 10. Blocage par catégorie (réseaux sociaux)

Créez un alias pour regrouper les domaines de réseaux sociaux, puis une règle de blocage.

![img1](image-TP5/img21.png)
![img1](image-TP5/img22.png)

Analysez les logs pour voir les tentatives bloquées :

![img1](image-TP5/img23.png)
![img1](image-TP5/img24.png)

**Question :**

- **Que se passe‑t‑il si la règle est placée sous une règle « Pass Any » ?**  
  Elle ne sera jamais appliquée. pfSense traite les règles de haut en bas et s’arrête à la première correspondance; la règle « Pass Any » laissera passer le trafic avant que le blocage soit évalué.

## 11. Règles horaires

Définissez un horaire :

![img1](image-TP5/img25.png)

Puis associez‑le à une règle existante :

![img1](image-TP5/img26.png)

**Questions :**

- **Pourquoi les règles horaires sont‑elles utiles en entreprise ?**  
  Elles permettent d’appliquer des restrictions adaptées aux plages de travail : limiter l’accès à certains services (réseaux sociaux, streaming) en dehors des heures ouvrables, automatiser la politique de sécurité et réduire les usages non professionnels.

## 12. Serveur web local

Installez un serveur web sur Ubuntu, puis configurez les règles pour n’autoriser que des accès spécifiques :

![img1](image-TP5/img27.png)
![img1](image-TP5/img28.png)

La règle autorisant uniquement l’IP source désignée vers le port 80 :

![img1](image-TP5/img29.png)

**Questions :**

- **Filtrer par IP source ?**  
  Oui. On limite l’accès à une machine précise afin de contrôler qui peut consulter le serveur.

- **Filtrer par port ?**  
  Oui. En ciblant le port 80, on protège uniquement le service HTTP, pas d’autres services évent
