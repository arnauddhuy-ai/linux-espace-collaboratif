# TP Linux : Collaboration et sécurité avancée des fichiers (SGID et ACL)

## 0. Prérequis matériels et logiciels

### 0.1 Machines nécessaires

**Configuration minimale :**
- **1 serveur Linux** (Debian 12, Ubuntu 22.04 LTS ou Rocky Linux 9)

**Configuration recommandée :**
- **1 serveur Linux** + **1 poste client** pour tests à distance (optionnel)

### 0.2 Spécifications techniques

| Machine | Système d'exploitation | RAM | Disque dur | Processeur |
|---------|------------------------|-----|------------|------------|
| **Serveur Linux** | Debian 12 / Ubuntu 22.04 | 2 GB | 20 GB | 1-2 vCPU |
| **Client** (optionnel) | Linux / Windows | 1 GB | 20 GB | 1 vCPU |

### 0.3 Logiciels requis

- **Hyperviseur** : VMware Workstation, VirtualBox, ou KVM
- **Distribution Linux** : Debian 12, Ubuntu Server 22.04 LTS ou Rocky Linux 9
- **Paquet ACL** : sera installé durant le TP

### 0.4 État initial requis

- Linux installé avec accès root ou sudo
- Connexion réseau fonctionnelle (pour apt/dnf)
- Accès SSH configuré (recommandé)

### 0.5 Durée estimée

**1 à 2 heures** (incluant tests et vérifications)

---

## 1. Introduction

Dans un environnement professionnel, plusieurs équipes (développeurs, testeurs, administrateurs) doivent souvent collaborer sur les mêmes ressources.

Cependant, chaque groupe doit avoir des **droits d'accès adaptés** à ses besoins, pour garantir à la fois la **productivité** et la **sécurité des données**.

Sous Linux, cette gestion s'appuie sur plusieurs mécanismes :

- **Les utilisateurs** : chaque personne possède un compte individuel
- **Les groupes** : permettent de mutualiser des ressources entre plusieurs utilisateurs
- **Les permissions, le bit SGID et les ACL** : définissent qui peut lire, écrire ou exécuter un fichier, et assurent un héritage automatique et précis des droits d'accès

> Ce TP vous guidera pas à pas pour mettre en place un **espace collaboratif multi-groupes**, sécurisé et fonctionnel, en utilisant les **ACL (Access Control Lists)** et le **bit SGID**.

---

## 2. Objectifs pédagogiques

À l'issue de ce TP, vous serez capable de :

- Créer et gérer des **utilisateurs** et des **groupes** sous Linux
- Configurer deux répertoires partagés distincts pour deux groupes de travail
- Mettre en œuvre le **bit SGID** et les **ACL** pour un héritage automatique des droits
- Tester les droits d'accès entre plusieurs utilisateurs
- Comprendre l'intérêt des ACL dans un contexte professionnel

---

## 3. Préparation et gestion des utilisateurs / groupes

### 3.1 Création des utilisateurs

```bash
sudo useradd -m julien
sudo passwd julien

sudo useradd -m marie
sudo passwd marie

sudo useradd -m amine
sudo passwd amine

sudo useradd -m lea
sudo passwd lea
```

### Explication :

- `useradd -m` crée l'utilisateur et son répertoire personnel
- `passwd` définit le mot de passe

### 3.2 Création des deux groupes de travail

```bash
sudo groupadd devweb
sudo groupadd testweb
```

### 3.3 Ajout des utilisateurs dans les groupes

```bash
sudo usermod -aG devweb julien
sudo usermod -aG devweb marie

sudo usermod -aG testweb amine
sudo usermod -aG testweb lea
```

### 3.4 Vérification des appartenances

```bash
grep 'devweb' /etc/group
grep 'testweb' /etc/group
```
**Capture :**

![Test 1](./01.%20Vérification%20des%20groupes.png)

Vérification de la création des groupes devweb et testweb et de l'affectation des utilisateurs.

### Résultat attendu :

```
devweb:x:1005:julien,marie
testweb:x:1006:amine,lea
```

> Permet de s'assurer que les utilisateurs sont bien rattachés à leurs groupes.

---

## 4. Installation et préparation du système ACL

Les ACL ne sont pas toujours activées par défaut.

### Installation du paquet ACL :

```bash
sudo apt update
sudo apt install acl -y
```

> **Note :** Sur Rocky Linux / CentOS, utilisez `sudo dnf install acl -y`

---

## 5. Création et configuration des répertoires partagés

### 5.1 Création des deux dossiers collaboratifs

```bash
sudo mkdir -p /srv/ProjetWeb/DevApp
sudo mkdir -p /srv/ProjetWeb/TestApp
```

### 5.2 Attribution des groupes propriétaires

```bash
sudo chown :devweb /srv/ProjetWeb/DevApp
sudo chown :testweb /srv/ProjetWeb/TestApp
```

### 5.3 Application des permissions de base et du bit SGID

```bash
sudo chmod 2770 /srv/ProjetWeb/DevApp
sudo chmod 2770 /srv/ProjetWeb/TestApp
```

### Explication :

- **770** → accès total pour le propriétaire et le groupe, aucun pour les autres
- **2 (SGID)** → assure que les nouveaux fichiers héritent du groupe du dossier

---

## 6. Application des ACL pour les droits par défaut

### 6.1 Configuration des ACL pour DevApp

```bash
sudo setfacl -R -m g:devweb:rwx /srv/ProjetWeb/DevApp
sudo setfacl -R -m d:g:devweb:rwx /srv/ProjetWeb/DevApp
sudo setfacl -R -m d:m::rwx /srv/ProjetWeb/DevApp
```

### 6.2 Configuration des ACL pour TestApp

```bash
sudo setfacl -R -m g:testweb:rwx /srv/ProjetWeb/TestApp
sudo setfacl -R -m d:g:testweb:rwx /srv/ProjetWeb/TestApp
sudo setfacl -R -m d:m::rwx /srv/ProjetWeb/TestApp
```

### Explication des options :

- `-R` : applique récursivement aux fichiers existants
- `-m` : modifie les ACL
- `g:devweb:rwx` : donne les droits rwx au groupe devweb
- `d:g:devweb:rwx` : définit les droits par défaut pour les futurs fichiers
- `d:m::rwx` : définit le masque par défaut

### 6.3 Vérification des ACL

```bash
getfacl /srv/ProjetWeb/DevApp
getfacl /srv/ProjetWeb/TestApp
```
**Capture :**

![Test 2](./02.%20Vérification%20des%20ACL%20sur%20DevApp.png)

Affichage détaillé des droits d'accès et des règles d'héritage par défaut pour le répertoire DevApp.

### Résultat attendu :

```
getfacl : suppression du premier << /  >> des noms de chemins absolus
# file: srv/ProjetWeb/DevApp
# owner: root
# group: devweb
# flags: -s-
user::rwx
group::rwx
other::---
default:user::rwx
default:group::rwx
default:mask::rwx
default:other::---
```

![Test 3](./03.%20Vérification%20des%20ACL%20sur%20TestApp.png)

Affichage détaillé des droits d'accès et des règles d'héritage par défaut pour le répertoire TestApp.

### Résultat attendu :

```
getfacl : suppression du premier << /  >> des noms de chemins absolus
# file: srv/ProjetWeb/testApp
# owner: root
# group: testweb
# flags: -s-
user::rwx
group::rwx
other::---

```

### 6.4 Vérification supplémentaire

```bash
ls -ld /srv/ProjetWeb/*
```
**Capture :**

![Test 4](./04.%20Vérification%20des%20permissions%20et%20SGID%20des%20dossiers.png)

Confirmation de l'activation du bit SGID (s) et de la présence des ACL (+) sur les répertoires partagés.

### Résultat attendu :

```
drwxrws---+ 2 root devweb  4096 20 Janv. 15:42 /srv/ProjetWeb/DevApp
drwxrws---  2 root testweb 4096 20 Janv. 15:53 /srv/ProjetWeb/TestApp
```

---

## 7. Tests collaboratifs

### 7.1 Test 1 : Collaboration dans le groupe devweb

#### Étape 1 : Julien crée un fichier

```bash
su - julien
cd /srv/ProjetWeb/DevApp
touch index.html
echo "<h1>Page créée par Julien</h1>" > index.html
ls -l
```
**Capture :**

![Test 5](./05.%20Test%20collaboration%20dans%20DevApp%20(Julien%20crée%20un%20fichier).png)

Vérification de l'attribution automatique du groupe propriétaire devweb lors de la création d'un fichier par Julien.

### Résultat attendu :

```
-rw-rw----+ 1 julien devweb 33 Janv. 20 16:17 index.html
```

> Le groupe doit être `devweb` grâce au bit SGID

```bash
exit
```

#### Étape 2 : Marie modifie le même fichier

```bash
su - marie
cd /srv/ProjetWeb/DevApp
echo "<p>Ajout par Marie</p>" >> index.html
cat index.html
```
**Capture :**

![Test 6](./06.%20Test%20collaboration%20dans%20DevApp%20(Marie%20modifie%20le%20fichier).png)

Validation du travail collaboratif : Marie parvient à modifier le fichier créé par Julien grâce aux permissions de groupe.

### Résultat attendu :

```html
<h1>Page créée par Julien</h1>
<p>Ajout par Marie</p>
```

```bash
exit
```

#### En cas d'erreur "Permission denied"

Si Marie ne peut pas modifier le fichier :

```bash
sudo chmod g+w /srv/ProjetWeb/DevApp/index.html
```

Puis revérifier avec `ls -l`.

---

### 7.2 Test 2 : Isolation entre groupes

#### Étape 1 : Amine tente d'accéder à DevApp

```bash
su - amine
cd /srv/ProjetWeb/DevApp
ls
```
**Capture :**

![Test 7](./07.%20Test%20isolation%20entre%20groupes%20(Amine%20tente%20d’accéder%20à%20DevApp).png)

Test de sécurité réussi : l'accès au répertoire DevApp est refusé à l'utilisateur Amine (membre de TestApp).

### Résultat attendu :

```
-sh: l: cd: can't cd to /srv/ProjetWeb/DevApp
```

> Amine **ne doit pas** pouvoir accéder au dossier `DevApp`, car il n'est pas membre du groupe devweb

```bash
exit
```

#### Étape 2 : Léa travaille dans TestApp

```bash
su - lea
cd /srv/ProjetWeb/TestApp
touch rapport.txt
echo "Rapport de test par Léa" > rapport.txt
cat rapport.txt
ls -l
```

### Résultat attendu :

```
Rapport de test par Léa
-rw-rw-r-- 1 lea testweb 24 Jan  9 10:20 rapport.txt
```

> Léa appartient au groupe `testweb` et peut donc créer et modifier des fichiers dans `TestApp`

```bash
exit
```

---

### 7.3 Test 3 : Vérification croisée

#### Amine crée un fichier dans TestApp

```bash
su - amine
cd /srv/ProjetWeb/TestApp
touch test_amine.txt
echo "Test effectué par Amine" > test_amine.txt
exit
```

#### Léa modifie le fichier d'Amine

```bash
su - lea
cd /srv/ProjetWeb/TestApp
echo "Validation par Léa" >> test_amine.txt
cat test_amine.txt
exit
```

**Capture :**

![Test 8](./08.%20Test%20collaboration%20dans%20TestApp%20(Amine%20et%20Léa).png)

Confirmation finale de la collaboration fluide entre Amine et Léa dans leur espace partagé dédié.

### Résultat attendu :

```
Test effectué par Amine
Validation par Léa
```

> Les deux membres du groupe `testweb` peuvent collaborer sur les mêmes fichiers

---

## 8. Synthèse des mécanismes utilisés

### 8.1 Comparaison des technologies

| Mécanisme | Rôle | Avantages | Limites |
|-----------|------|-----------|---------|
| **Permissions Unix standard** | Droits de base (rwx) | Simple, universel | 1 seul groupe par fichier |
| **Bit SGID** | Héritage du groupe | Automatique pour nouveaux fichiers | Ne gère pas les permissions fines |
| **ACL** | Permissions avancées | Plusieurs groupes/utilisateurs | Plus complexe à gérer |

### 8.2 Commandes essentielles

| Commande | Usage |
|----------|-------|
| `setfacl -m g:groupe:rwx fichier` | Ajouter des droits ACL |
| `setfacl -m d:g:groupe:rwx dossier` | Définir ACL par défaut |
| `getfacl fichier` | Afficher les ACL |
| `setfacl -b fichier` | Supprimer toutes les ACL |
| `ls -l` | Vérifier permissions (+ indique ACL) |
| `chmod 2770` | Appliquer SGID + permissions |

---

## 9. Dépannage rapide

| Problème | Cause probable | Solution |
|----------|----------------|----------|
| **Permission denied** lors de la modification | Permissions fichier incorrectes | `sudo chmod g+w fichier` |
| **Groupe incorrect** sur nouveau fichier | SGID non appliqué | Vérifier `chmod 2770` |
| **ACL non héritées** | ACL par défaut manquantes | Vérifier `setfacl -m d:g:...` |
| **getfacl command not found** | Paquet ACL non installé | `sudo apt install acl` |
| **Utilisateur ne peut pas changer de dossier** | Pas dans le bon groupe | Vérifier `groups utilisateur` |

---

## Conclusion

Ce TP vous a permis de maîtriser la gestion avancée des droits d'accès sous Linux en combinant :

- **Groupes Unix** pour organiser les utilisateurs
- **Bit SGID** pour l'héritage automatique du groupe
- **ACL** pour des permissions granulaires et évolutives

Ces compétences sont essentielles pour administrer des serveurs Linux en production et garantir à la fois la collaboration et la sécurité des données dans un environnement professionnel.
