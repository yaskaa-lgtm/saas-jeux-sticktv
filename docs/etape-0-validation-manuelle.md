# Étape 0 : plan de validation manuel sur la H96 Max

Objectif : vérifier, **sans écrire de code applicatif**, que la chaîne matérielle et logicielle
tient la route avant d'y investir du développement. Chaque test indique **quoi faire**, **ce qu'il
faut observer** et **ce qu'il faut me renvoyer**. Le gabarit de compte-rendu est à la fin du
document (§J).

> Convention : remplace `192.168.1.50` par l'IP réelle de la box dans toutes les commandes.

---

## Prérequis

- La H96 Max branchée sur la TV, et si possible **aussi un câble Ethernet** : la box a un port RJ45
  et on comparera WiFi et câble.
- La manette Bluetooth : modèle exact, et **batterie rechargeable** de préférence.
- Un PC (ou le futur Raspberry Pi) sur le **même réseau local**, avec Android platform-tools
  installés (`adb version` doit répondre).
- Un smartphone et un chronomètre.
- Un **nouveau** compte Microsoft (voir le test B), jamais ton compte perso.
- Idéalement, réaliser les tests C, F et H **dans le bar, aux heures d'affluence**. Le WiFi d'un bar
  plein un vendredi soir n'a rien à voir avec celui de chez toi.

---

## A. Préparation de la box et ADB par le réseau

1. Relève le modèle exact, la version d'Android et la version du firmware (Paramètres → À propos).
2. Active les options développeur (7 clics sur « Numéro de build »), puis le **débogage USB**, et
   le **débogage ADB par le réseau** s'il est proposé.
3. Sur la box : désactive la mise en veille, l'écran de veille et l'économiseur d'énergie
   (« Jamais »).
4. Sur la box : **réservation DHCP** dans la box internet, pour que l'IP de la H96 Max ne change
   jamais.
5. Sur le PC : connecte-toi à la box, puis accepte sur la TV la fenêtre « Autoriser le débogage »
   en cochant **« Toujours autoriser depuis cet ordinateur »** :

```bash
adb connect 192.168.1.50:5555
```

```bash
adb devices
```

```bash
adb -s 192.168.1.50:5555 shell getprop ro.build.version.release
```

6. **Test de redémarrage** : redémarre la box, attends 2 min, puis lance `adb connect` à nouveau.
   - Si ça répond : ADB réseau persiste après un reboot, c'est le cas idéal.
   - Si ça ne répond pas : il faudra relancer `adb tcpip 5555` en USB après chaque reboot.
     C'est **rédhibitoire** pour un bar, à me signaler tout de suite.

**À me renvoyer** : la sortie de `adb devices`, la version d'Android, et le résultat du test de
redémarrage.

---

## B. Compte Microsoft dédié au kiosque

1. Crée une **nouvelle adresse e-mail** dédiée et un **nouveau compte Microsoft**, sans nom réel ni
   numéro de téléphone perso si possible.
2. **Aucun moyen de paiement** enregistré. Paie l'abonnement Game Pass avec une **carte ou un code
   prépayé**.
3. Vérifie **quelle formule inclut le cloud gaming en France** aujourd'hui et son prix exact.
4. Active la **validation en deux étapes** avec une application d'authentification sur **ton**
   téléphone, et conserve les codes de récupération hors ligne.
5. Paramètres de confidentialité Xbox (xbox.com → Confidentialité et sécurité en ligne) : messages,
   chat, invitations et partage de profil sur **« Bloquer »** ou **« Personne »**. Gamertag neutre,
   sans rapport avec toi.
6. Active **« Demander un mot de passe pour les achats »**.
7. **Tests de persistance de connexion** sur la box :
   - le compte est-il toujours connecté après `am force-stop` d'Edge (test D) ?
   - après un redémarrage de la box ?
   - après 24 h ? après 7 jours ?
   - Microsoft redemande-t-il une vérification d'identité (« Vérifiez que c'est bien vous ») ?
8. **Test d'achat** : dans un jeu ou dans le store xCloud, tente un achat. Il doit être **refusé ou
   bloqué**.

**À me renvoyer** : formule et prix, résultat de chaque test de persistance, résultat du test
d'achat.

---

## C. Qualité de Xbox Cloud Gaming sur la box

1. Installe Edge sur la box. Si tu dois le sideloader, prends l'APK depuis une **source fiable** et
   vérifie sa signature (le certificat doit être celui de Microsoft Corporation) :

```bash
apksigner verify --print-certs edge.apk
```

```bash
adb -s 192.168.1.50:5555 install edge.apk
```

2. Récupère le nom exact du package et de l'activité de démarrage. Je m'attends à
   `com.microsoft.emmx`, à confirmer :

```bash
adb -s 192.168.1.50:5555 shell pm list packages | grep -i -E "emmx|edge|chrome|browser"
```

```bash
adb -s 192.168.1.50:5555 shell cmd package resolve-activity --brief com.microsoft.emmx
```

3. Lance Edge à la main. **Valide une fois pour toutes** tous les écrans de premier lancement :
   connexion à Edge (refuser), notifications (refuser), « navigateur par défaut », etc.
4. Va sur `https://www.xbox.com/fr-FR/play` et connecte le compte kiosque. Note les points
   suivants :
   - un message « navigateur non pris en charge » apparaît-il ?
   - la manette est-elle reconnue ?
   - la page passe-t-elle en **plein écran**, ou la barre d'adresse reste-t-elle visible ?
5. Lance **3 jeux différents** (un jeu de course, un de combat, un de sport) et joue **15 min
   chacun**. Note pour chacun :
   - le temps entre le clic sur « Jouer » et le jeu jouable (chronomètre) ;
   - la file d'attente xCloud (oui/non, durée) ;
   - la résolution et la netteté, l'input lag ressenti (noté de 1 à 5), et les saccades ou
     déconnexions ;
   - si l'option existe, affiche les **statistiques du flux** dans les paramètres xCloud (débit,
     latence, pertes de paquets).
6. Fais une **comparaison réseau** : WiFi 2,4 GHz, WiFi 5 GHz, Ethernet. Mesure aussi le débit
   depuis la box (fast.com dans Edge). Microsoft recommande au moins 20 Mb/s et du 5 GHz ou de
   l'Ethernet.
7. Sur la TV : active le **mode « Jeu »** pour réduire la latence d'affichage.
8. **Test thermique** : 2 h de jeu continu. Note si la box chauffe fort ou ralentit, et si l'image
   se dégrade au fil du temps.

**À me renvoyer** : un tableau par jeu, la comparaison réseau et le résultat du test thermique.

---

## D. Lancement d'un jeu précis par URL via ADB

1. Sur un PC, ouvre xbox.com/play, va sur la page d'un jeu et copie son URL. Teste les **deux
   formes** :
   - `https://www.xbox.com/fr-FR/play/games/<slug>/<productId>` (page du jeu)
   - `https://www.xbox.com/fr-FR/play/launch/<slug>/<productId>` (lancement direct, à confirmer)
2. Lance avec le package Edge forcé :

```bash
adb -s 192.168.1.50:5555 shell am start -a android.intent.action.VIEW -d "URL_DU_JEU" -p com.microsoft.emmx
```

3. Observe et note :
   - le délai entre la commande et la page affichée, puis entre la commande et le jeu jouable ;
   - si le jeu **démarre tout seul** ou s'il faut appuyer sur **« Jouer »** à la manette. C'est
     crucial pour le démarrage du timer (voir le point I1 de `00-risques-et-points-a-trancher.md`) ;
   - si chaque commande ouvre un **nouvel onglet**, et si les onglets s'accumulent ;
   - ce qui se passe si on lance un 2e jeu alors qu'un 1er tourne.
4. **Vérification du premier plan.** Copie-moi les sorties **brutes** de ces deux commandes avec
   Edge au premier plan, puis à nouveau avec le launcher Android au premier plan (je m'en servirai
   pour écrire le parseur de l'Étape 2) :

```bash
adb -s 192.168.1.50:5555 shell dumpsys window | grep -E "mCurrentFocus|mFocusedApp"
```

```bash
adb -s 192.168.1.50:5555 shell dumpsys activity activities | grep -E "mResumedActivity|topResumedActivity"
```

5. **Fermeture** :

```bash
adb -s 192.168.1.50:5555 shell am force-stop com.microsoft.emmx
```

   Puis relance Edge sur une URL neutre :

```bash
adb -s 192.168.1.50:5555 shell am start -a android.intent.action.VIEW -d "https://example.com" -p com.microsoft.emmx
```

   Note :
   - Edge **restaure-t-il l'onglet du jeu** précédent ?
   - le compte est-il **toujours connecté** sur xbox.com/play ?
6. Relance **immédiatement** un autre jeu après le `force-stop`. Un message « Vous jouez déjà sur
   un autre appareil » ou un délai apparaît-il ?

**À me renvoyer** : les deux formes d'URL testées et leur comportement, les sorties brutes de
`dumpsys`, la restauration d'onglets (oui/non) et la persistance de connexion (oui/non).

---

## E. Comportement de la manette

1. Associe la manette. Note le **modèle exact** et son nom tel qu'il apparaît sur Android.
2. **Détection.** Copie-moi les sorties de cette commande **manette connectée**, puis **manette
   éteinte** :

```bash
adb -s 192.168.1.50:5555 shell dumpsys input | grep -i -E -A2 "xbox|gamepad|controller|joystick"
```

3. **Batterie** : vérifie si le niveau de batterie remonte. Ce n'est pas garanti sur Android 11.
   Copie-moi la sortie, même vide :

```bash
adb -s 192.168.1.50:5555 shell dumpsys bluetooth_manager | grep -i -E "battery|level"
```

4. **Reconnexion** :
   - après un reboot de la box, la manette se reconnecte-t-elle seule ?
   - après la mise en veille automatique de la manette (environ 15 min sans action), un appui sur le
     bouton central suffit-il ?
5. **Boutons dangereux**, pour le kiosque : que fait chaque bouton **pendant un jeu** et **dans
   Edge** ? Est-ce qu'il fait sortir du navigateur ou ouvre un menu Android ?
   - bouton Xbox (central) ;
   - bouton « View » / « Menu » ;
   - appui long sur le bouton central.
6. **Mapping** : A, B, X, Y et les gâchettes sont-ils corrects dans les 3 jeux ?

**À me renvoyer** : les sorties `dumpsys` dans les deux états, le résultat batterie, la reconnexion
et le comportement de chaque bouton.

---

## F. Stabilité d'ADB par le WiFi sur plusieurs heures

Lance ce script sur le PC ou le Raspberry Pi **au moins 8 h**, idéalement une nuit **plus** une
soirée d'ouverture du bar. Il teste ADB toutes les 30 s et tente une reconnexion en cas d'échec.

**Linux, macOS ou Raspberry Pi (bash)** :

```bash
IP=192.168.1.50:5555; while true; do if timeout 10 adb -s $IP shell echo ok 2>/dev/null | grep -q ok; then R=OK; else R=KO; timeout 10 adb connect $IP >/dev/null 2>&1; fi; echo "$(date '+%F %T') $R" | tee -a adb-stabilite.log; sleep 30; done
```

**Windows (PowerShell)** :

```powershell
$ip="192.168.1.50:5555"; while ($true) { $o = adb -s $ip shell echo ok 2>&1; if ("$o" -match "ok") { $r="OK" } else { $r="KO"; adb connect $ip | Out-Null }; "$(Get-Date -Format s) $r" | Tee-Object -FilePath adb-stabilite.log -Append; Start-Sleep 30 }
```

Bilan, nombre d'échecs :

```bash
grep -c KO adb-stabilite.log
```

Tests de perturbation à faire pendant que le script tourne :
1. Éteindre et rallumer la **box** : combien de temps avant le retour à `OK` ?
2. Redémarrer le **routeur** : retour automatique à `OK` ?
3. Laisser la box **inactive 1 h**, écran sur l'accueil Android : ADB reste-t-il joignable ?

**À me renvoyer** : le fichier `adb-stabilite.log` (ou le nombre de KO et leurs horaires) et le
résultat des 3 perturbations.

---

## G. Verrouillage kiosque

Objectif : lister **toutes les façons** dont un client pourrait sortir du jeu ou du navigateur.
1. Vérifie si la box propose **l'épinglage d'écran** (Paramètres → Sécurité → Épinglage d'écran).
   Il est souvent absent sur Android TV.
2. Avec la manette **et** sans la télécommande (le client n'y aura pas accès), essaie de :
   - revenir au launcher Android ;
   - ouvrir les Paramètres Android ;
   - ouvrir une autre application ;
   - dans Edge, ouvrir un nouvel onglet ou taper une adresse ;
   - aller sur `account.microsoft.com`.
3. Vérifie si la box a des **comptes Google** configurés (Paramètres → Comptes). C'est important
   pour l'option « device owner » de verrouillage, qui demande une box sans compte :

```bash
adb -s 192.168.1.50:5555 shell dumpsys account | grep -i "Account {"
```

4. Note si l'application **Fully Kiosk Browser** s'installe et s'affiche correctement (version
   d'essai gratuite). Elle servira à afficher la page TV.

**À me renvoyer** : la liste des « sorties » possibles, la présence ou non de l'épinglage et des
comptes Google, et le résultat de Fully Kiosk.

> La documentation complète du verrouillage (launcher kiosque, device owner, désactivation des
> boutons) viendra à l'Étape 8, adaptée à ce que tu auras observé ici.

---

## H. Réseau du bar

1. Le WiFi du bar a-t-il une **isolation des clients** (réseau invité) ? Si oui, l'agent et la box
   **ne se verront pas**. Il faudra soit un réseau privé (pas le WiFi client), soit un petit routeur
   dédié.
2. Le réseau laisse-t-il sortir le **HTTPS (port 443)** ? Il le faut pour que l'agent se connecte
   au backend.
3. Débit et latence **depuis la box**, à 18 h puis à 22 h un jour d'affluence.
4. Faut-il un **routeur ou point d'accès dédié** à la box et au Raspberry Pi ? C'est souvent la
   solution la plus fiable.

---

## I. Catalogue et URL des jeux

1. Choisis **5 à 10 jeux** disponibles **en cloud** dans le Game Pass, adaptés à un bar : parties
   courtes, jouables à 1 manette, PEGI raisonnable.
2. Pour chacun, note le **titre**, **l'URL** qui fonctionne (test D), la **classification PEGI**,
   le temps de chargement et s'il y a une **progression sauvegardée** (voir le point I2 de
   `00-risques-et-points-a-trancher.md`).
3. Attention : le catalogue Game Pass **tourne**, et des jeux le quittent chaque mois. Il faudra
   surveiller cela dans le back-office.

---

## Checklist légale (à vérifier, ce n'est pas un avis juridique)

> Je ne donne pas d'avis juridique définitif. Chaque point est **à faire vérifier** par un
> professionnel (avocat, expert-comptable, CCI) avant l'ouverture au public.

**Microsoft, Xbox et éditeurs**
- [ ] Contrat de services Microsoft, conditions d'utilisation Xbox et conditions Game Pass :
  clause « usage personnel et non commercial », et possibilité d'une **autorisation écrite** de
  Microsoft pour un usage en établissement.
- [ ] Contrats de licence (EULA) des éditeurs des jeux choisis : diffusion ou usage public.
- [ ] Risque de **suspension du compte** et ce qu'on perd dans ce cas (abonnement prépayé).

**Clients et âge**
- [ ] Obligations liées à la **signalétique PEGI** et à la mise à disposition de jeux 16+ ou 18+ à
  des mineurs dans un lieu public.
- [ ] Règles d'**accueil des mineurs dans un débit de boissons** selon la licence du bar (Code de
  la santé publique).
- [ ] **Avertissement épilepsie et photosensibilité** obligatoire pour les jeux vidéo : affichage
  sur la page mobile.
- [ ] Aucun **gain** ni lot attaché aux parties : sinon on bascule potentiellement dans la
  réglementation des jeux d'argent ou des loteries. Attention aux « tournois payants avec lots ».

**Vente et consommation**
- [ ] **CGV** et **mentions légales** accessibles depuis la page mobile.
- [ ] **Droit de rétractation** : exception pour un service pleinement exécuté avec accord exprès du
  consommateur. Prévoir la case ou la mention adéquate.
- [ ] **Médiateur de la consommation** : obligation de désigner un médiateur pour la vente aux
  particuliers.
- [ ] **Justificatif ou ticket** : SumUp envoie-t-il un reçu ? Faut-il une facture sur demande ?

**Entreprise et fiscalité**
- [ ] Statut juridique, immatriculation, **TVA** (franchise ou 20 %) sur la prestation.
- [ ] **Contrat avec le bar** : partage des revenus, électricité, internet, responsabilité du
  matériel (vol, casse), assurance.
- [ ] **Assurance RC professionnelle**.
- [ ] Diffusion du **son des jeux** dans un lieu public : vérifier si le contrat SACEM / SPRE du bar
  couvre ce cas.

**Données personnelles (RGPD)**
- [ ] Données minimales : pas de compte client, pas de nom. Seulement l'ID de session, des
  horodatages et l'IP dans les logs.
- [ ] **Politique de confidentialité**, durée de conservation des logs, statut de SumUp vis-à-vis
  des données de paiement.
- [ ] Pas de cookie hors cookies techniques, donc a priori pas de bandeau. À confirmer.

---

## J. Gabarit de compte-rendu à me renvoyer

```
A. ADB réseau
   Android / firmware : …
   adb devices : …
   Persiste après reboot : oui / non

B. Compte Microsoft
   Formule Game Pass + prix : …
   Connecté après force-stop / reboot / 24 h / 7 j : … / … / … / …
   Achat bloqué : oui / non

C. Qualité xCloud
   « Navigateur non supporté » : oui / non     Plein écran auto : oui / non
   Jeu 1 : … | chargement … s | file … | qualité …/5 | lag …/5 | coupures …
   Jeu 2 : …
   Jeu 3 : …
   Débit box WiFi 2,4 / 5 GHz / Ethernet : … / … / … Mb/s
   Thermique 2 h : …

D. Lancement par URL
   Package Edge + activité : …
   /play/games/... : démarre seul / faut appuyer « Jouer »
   /play/launch/... : démarre seul / faut appuyer « Jouer » / ne marche pas
   Nouvel onglet à chaque lancement : oui / non
   Onglet restauré après force-stop : oui / non
   « Vous jouez déjà ailleurs » : oui / non
   Sorties dumpsys (Edge / launcher) : [coller]

E. Manette
   Modèle : …   Reconnexion après reboot / veille : … / …
   dumpsys input connectée / éteinte : [coller]
   Batterie visible : oui / non   [coller]
   Bouton Xbox / View / Menu : …

F. Stabilité ADB
   Durée : … h   Nombre de KO : …   Horaires : …
   Retour après reboot box / routeur : … s / … s

G. Kiosque
   Épinglage dispo : oui / non   Comptes Google : …
   Sorties possibles : …
   Fully Kiosk OK : oui / non

H. Réseau du bar
   Isolation clients : oui / non   443 sortant OK : oui / non
   Débit 18 h / 22 h : … / …

I. Jeux retenus (titre | URL | PEGI | chargement | sauvegarde)
   …

Réponses aux 8 questions de 00-risques-et-points-a-trancher.md §5 :
   1. …  2. …  3. …  4. …  5. …  6. …  7. …  8. …
```

---

## Critères de go / no-go pour passer à l'Étape 1

| Critère | Seuil |
|---|---|
| ADB réseau après reboot | doit revenir **sans USB** |
| Stabilité ADB sur 8 h | KO isolés uniquement, avec reconnexion automatique |
| Qualité xCloud | au moins 3/5 en qualité et en lag, aux heures d'affluence |
| Lancement par URL | au moins une forme d'URL ouvre le bon jeu |
| Compte connecté après `force-stop` | **obligatoire** |
| Point juridique R1 | au moins un premier avis obtenu, ou risque accepté en connaissance de cause |
