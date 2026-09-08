---
title: "Bloquer la pub sur tout son ordinateur avec DNSforge"
description: "Un résolveur DNS allemand, gratuit et sans journalisation, qui filtre publicités, traqueurs et logiciels malveillants. Pas à pas pour Windows et macOS."
pubDate: 2026-09-08
tags: ["dns", "securite", "windows", "mac"]
---

Un bloqueur de publicité classique vit dans le navigateur. Il ne voit donc rien de ce qui se passe ailleurs : les publicités d'une application mobile, les statistiques que votre téléviseur connecté envoie la nuit, les traqueurs intégrés à un logiciel de bureau. Filtrer au niveau du DNS règle le problème un cran plus bas, pour tout l'appareil d'un coup.

[DNSforge](https://dnsforge.de/) fait exactement cela, gratuitement. Voici comment le mettre en place sur Windows et sur Mac, et surtout comment vérifier que ça fonctionne vraiment.

## Le DNS en deux minutes

Le DNS est l'annuaire d'Internet. Quand vous tapez `exemple.fr`, votre ordinateur ne sait pas où aller : il demande à un serveur DNS de traduire ce nom en adresse numérique. Cette question, il la pose des centaines de fois par jour, pour chaque image, chaque script, chaque mouchard d'une page.

Par défaut, c'est votre fournisseur d'accès qui répond. Un résolveur filtrant comme DNSforge se contente d'une chose : quand la question porte sur un domaine publicitaire ou un traqueur connu, il refuse de répondre. Le contenu n'est jamais téléchargé, donc jamais affiché.

Trois conséquences agréables : les pages se chargent plus vite, la bande passante diminue, et le filtrage s'applique à **tous** les logiciels de la machine, pas seulement au navigateur.

Une limite honnête, en revanche : quand une publicité est servie depuis le même domaine que le contenu, c'est le cas sur YouTube, le DNS ne peut rien faire sans casser le site entier. Le filtrage DNS complète un bloqueur de navigateur, il ne le remplace pas.

## Qui est derrière DNSforge

Le service est opéré par Giebel.IT, l'équipe allemande derrière adminForge. Les points qui comptent :

- **Serveurs exclusivement en Allemagne**, donc sous RGPD
- **Aucune journalisation** des requêtes
- **DNSSEC validé** : les réponses falsifiées sont rejetées
- **Chiffrement disponible** en DoT, DoH et DoQ
- **Gratuit**, financé par les dons
- Limite de 100 requêtes toutes les 10 secondes, largement suffisant pour un foyer

Les listes de blocage proviennent de sources publiques réputées (hagezi, OISD, StevenBlack, AdGuard, BlocklistProject) et sont mises à jour automatiquement chaque jour.

## Choisir sa variante

DNSforge propose quatre résolveurs distincts. Le choix se fait à l'adresse près.

| Variante | Pubs | Traqueurs | Malwares | Adulte | Jeux d'argent |
|----------|------|-----------|----------|--------|---------------|
| **Normal** | ✅ | ✅ | ✅ | ❌ | ❌ |
| **Clean** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Hard** | ✅ | ✅ | ✅ | ❌ | ❌ |
| **Blank** | ❌ | ❌ | ❌ | ❌ | ❌ |

**Normal** (1,4 million de domaines) est l'équilibre recommandé pour commencer : agressif sur la publicité, prudent sur le reste. **Clean** (7,8 millions) ajoute un filtrage familial. **Hard** serre la vis sur le pistage au prix de quelques sites récalcitrants. **Blank** ne filtre rien : c'est un simple résolveur privé et chiffré.

Ce guide utilise **Normal**. Pour une autre variante, remplacez simplement les adresses par celles indiquées sur leur site.

## Les adresses (variante Normal)

```
IPv4 : 49.12.67.122
       91.99.154.175
       176.9.93.198
       176.9.1.117

DNS-over-HTTPS : https://dnsforge.de/dns-query
DNS-over-TLS   : dnsforge.de
```

Deux adresses IPv4 suffisent : la seconde sert de secours si la première ne répond pas.

## Windows 11 : la méthode simple

Cinq minutes, aucune ligne de commande. Le filtrage fonctionne, mais vos requêtes circulent en clair jusqu'aux serveurs allemands.

1. **Paramètres** → **Réseau et Internet**
2. Cliquez sur votre connexion : **Wi-Fi** ou **Ethernet**
3. **Propriétés du matériel**
4. En face de **Attribution de serveur DNS**, cliquez sur **Modifier**
5. Choisissez **Manuel** dans le menu déroulant
6. Activez l'interrupteur **IPv4**
7. **DNS préféré** : `49.12.67.122`
8. **Autre DNS** : `91.99.154.175`
9. **Enregistrer**

C'est immédiat, aucun redémarrage n'est nécessaire.

## Windows 11 : la méthode chiffrée

Windows 11 sait faire du DNS-over-HTTPS, mais uniquement avec une liste de fournisseurs qu'il connaît d'avance. DNSforge n'en fait pas partie : il faut donc le déclarer soi-même, une fois pour toutes.

Ouvrez **PowerShell en tant qu'administrateur** (clic droit sur le menu Démarrer → *Terminal (administrateur)*), puis collez ces deux commandes :

```powershell
Add-DnsClientDohServerAddress -ServerAddress "49.12.67.122" -DohTemplate "https://dnsforge.de/dns-query" -AllowFallbackToUdp $False -AutoUpgrade $True

Add-DnsClientDohServerAddress -ServerAddress "91.99.154.175" -DohTemplate "https://dnsforge.de/dns-query" -AllowFallbackToUdp $False -AutoUpgrade $True
```

`-AllowFallbackToUdp $False` est le réglage important : il interdit à Windows de retomber discrètement en DNS non chiffré si le chiffrement échoue. Mieux vaut une erreur visible qu'une fuite silencieuse.

Vérifiez que la déclaration a pris :

```powershell
Get-DnsClientDohServerAddress
```

Refaites ensuite les étapes 1 à 9 de la méthode simple. Un nouveau menu **Chiffrement DNS préféré** apparaît désormais sous chaque champ : choisissez **Chiffré uniquement (DNS sur HTTPS)**, puis enregistrez.

## Windows 10

Windows 10 ne gère pas le DNS chiffré de façon fiable. Contentez-vous des adresses classiques :

1. **Panneau de configuration** → **Réseau et Internet** → **Centre Réseau et partage**
2. **Modifier les paramètres de la carte**
3. Clic droit sur votre connexion → **Propriétés**
4. Sélectionnez **Protocole Internet version 4 (TCP/IPv4)** → **Propriétés**
5. Cochez **Utiliser l'adresse de serveur DNS suivante**
6. Serveur préféré : `49.12.67.122` / Serveur auxiliaire : `91.99.154.175`
7. **OK**, puis **Fermer**

## macOS : la méthode simple

1. **Réglages Système** → **Réseau**
2. Cliquez sur votre connexion active (**Wi-Fi** ou **Ethernet**)
3. Bouton **Détails…**
4. Onglet **DNS** dans la colonne de gauche
5. Sous *Serveurs DNS*, retirez les adresses existantes avec le bouton **−**
6. Ajoutez avec **+** : `49.12.67.122` puis `91.99.154.175`
7. **OK**

Attention : ce réglage est propre à chaque réseau. Votre Wi-Fi domestique et le partage de connexion de votre téléphone sont deux configurations distinctes.

## macOS — la méthode chiffrée

macOS ne propose aucune interface pour le DNS chiffré, mais il accepte les profils de configuration. DNSforge en publie un, pensé pour iOS, qui fonctionne aussi sur Mac (macOS 11 et suivants).

1. Téléchargez le profil : [dnsforge-doh.mobileconfig](https://dnsforge.de/dnsforge-doh.mobileconfig)
2. Double-cliquez sur le fichier téléchargé
3. Ouvrez **Réglages Système** → **Général** → **Gestion des appareils** (sur les versions plus anciennes : *Confidentialité et sécurité* → *Profils*)
4. Sélectionnez le profil `dnsforge` en attente, puis **Installer**
5. Confirmez avec votre mot de passe de session

Le profil prend le pas sur les réglages réseau et s'applique à tous vos réseaux d'un coup, ce qui est précisément l'intérêt. Pour l'annuler, sélectionnez-le au même endroit et cliquez sur **−**.

## Vérifier que ça marche vraiment

Ne sautez pas cette étape : un DNS mal appliqué est parfaitement silencieux.

**Le serveur utilisé.** Rendez-vous sur [dnscheck.tools](https://dnscheck.tools/). Les adresses affichées doivent appartenir à DNSforge, pas à votre fournisseur d'accès.

**Le filtrage.** Dans l'invite de commandes Windows, ou dans le Terminal sur Mac :

```
nslookup doubleclick.net
```

Vous devez obtenir une non-réponse : `0.0.0.0`, ou un message de type « serveur introuvable ». Si une véritable adresse publique s'affiche, le filtrage n'est pas actif, vos réglages ne sont pas appliqués, ou une autre configuration les court-circuite.

**Un doute sur un domaine précis.** L'outil *Domain Check* en bas de la page d'accueil de DNSforge dit si un domaine figure sur leurs listes.

## Quand un site casse

Ça arrive, rarement, et le symptôme est reconnaissable : une image ne charge pas, un bouton de paiement reste vide, une connexion échoue sans message clair.

Le réflexe : basculer temporairement sur **Blank** (`138.199.149.249`), ou revenir à votre configuration précédente, et recharger. Si le site fonctionne, un domaine légitime a été bloqué par erreur. DNSforge propose un formulaire de mise en liste blanche.

## Revenir en arrière

Sur Windows, reprenez le menu **Attribution de serveur DNS** et repassez sur **Automatique (DHCP)**. Sur Mac, supprimez les adresses ajoutées dans l'onglet DNS, ou retirez le profil. Aucune trace ne subsiste.

## Ce que ça ne fait pas

Pour finir, quelques illusions à dissiper. Le DNS chiffré protège vos **requêtes**, pas votre navigation :

- Il ne masque pas votre adresse IP. Ce n'est pas un VPN.
- Il ne chiffre pas le contenu de vos échanges, c'est le rôle du HTTPS.
- Le nom des sites visités reste souvent visible dans le trafic, par d'autres canaux.
- Votre fournisseur d'accès voit toujours à quelles adresses vous vous connectez.

Ce que vous y gagnez, concrètement : moins de publicité, moins de pistage, moins de bande passante gaspillée, et un annuaire qui ne consigne pas vos requêtes dans un fichier. C'est déjà beaucoup pour dix minutes de réglages et zéro euro.
