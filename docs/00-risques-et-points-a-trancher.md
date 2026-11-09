# Risques, incohérences et décisions à prendre (avant tout code)

Ce document liste ce que j'ai repéré en relisant le cahier des charges. Les points marqués
**[BLOQUANT]** peuvent remettre en cause le projet ou l'architecture et doivent être tranchés
pendant l'Étape 0. Les points **[DÉCISION]** demandent ta réponse avant l'Étape 1.

---

## 1. Risques bloquants

### R1 [BLOQUANT] Conditions d'utilisation Microsoft / Game Pass
Les conditions Xbox et Game Pass limitent l'usage à un **usage personnel et non commercial**
(formulation reprise dans plusieurs documents contractuels Game Pass ; texte officiel actuel à
relire toi-même). Faire payer des clients pour jouer sur un compte Game Pass est, a priori, un
usage commercial.
- Conséquences possibles : suspension ou bannissement du compte kiosque, perte de l'abonnement,
  voire réclamation de Microsoft ou des éditeurs.
- Ce n'est pas un avis juridique. Il faut le faire valider (avocat, ou demande d'autorisation écrite
  à Microsoft) **avant** d'investir dans le matériel de plusieurs bars.
- Impact technique : aucun. Le logiciel reste réutilisable pour d'autres contenus (jeux
  sous licence commerciale, émulation de jeux libres, jeux web).

### R2 [BLOQUANT] Xbox Cloud Gaming sur une box Android TV générique
Microsoft prend officiellement en charge xbox.com/play sur les navigateurs Android de téléphones et
tablettes. Une box Android TV non certifiée avec Edge installé à la main n'est pas une configuration
garantie : message « navigateur non pris en charge », résolution limitée à 720p, manette mal reconnue
ou mise à jour d'Edge qui casse tout.
- Plan B à garder en tête : un appareil avec l'**application Xbox officielle** (Fire TV Stick 4K /
  4K Max, ou une TV Samsung/LG récente). Le Fire TV accepte aussi ADB par le réseau, donc
  l'architecture reste la même. Il resterait à vérifier qu'on peut lancer un jeu précis par
  deep link dans l'application.

### R3 [BLOQUANT sécurité] Provenance de la H96 Max
Plusieurs box Android TV génériques vendues sur les marketplaces ont été livrées avec des firmwares
compromis (botnet BadBox / BadBox 2.0). Or cette box contiendra la session du compte Microsoft.
- À faire : vérifier le modèle et le firmware exacts, observer le trafic réseau si possible, ou
  partir directement sur un appareil de marque.
- Dans tous les cas, mettre la box sur un réseau qui ne voit pas la caisse du bar.

---

## 2. Incohérences ou zones floues du cahier des charges

### I1 « Le compte à rebours démarre quand le jeu est réellement lancé »
ADB permet seulement de savoir qu'**Edge est au premier plan**. Il ne voit pas si le flux cloud a
démarré (file d'attente xCloud, écran de connexion, bouton « Jouer » à presser, erreur). Il n'existe
pas de signal fiable « le jeu tourne ».
**Ma proposition** : le timer démarre au premier des deux événements suivants :
1. le client appuie sur **« C'est parti »** sur son téléphone ;
2. `LAUNCH_GRACE_SECONDS` (par exemple 90 s) après la confirmation qu'Edge est au premier plan.

Tu verras dans l'Étape 0 (test D) si l'URL lance directement le jeu ou s'il faut appuyer sur
« Jouer ». La réponse peut faire varier ce délai.

### I2 « Nettoyer la session cloud sans déconnecter le compte »
Effacer les cookies ou les données d'Edge (`pm clear`) **déconnecte** le compte Microsoft.
Le seul nettoyage sûr est `am force-stop` du navigateur. Deux risques à tester dans l'Étape 0 :
- Edge **restaure les onglets** au redémarrage, ce qui rouvrirait le jeu du client précédent ;
- la session cloud peut **continuer quelques minutes côté serveur** (« Vous jouez déjà sur un autre
  appareil »).

Autre point : tous les clients partagent les **sauvegardes** du même compte. Un client peut écraser
ou supprimer une sauvegarde. À accepter tel quel, ou à limiter en choisissant des jeux sans
progression (arcade, sport, combat).

### I3 Écran TV et jeu : un ou deux navigateurs ?
Si la page TV tourne dans Edge, le `force-stop` de fin de session la ferme aussi. Il faut alors
relancer Edge sur l'URL TV, et le problème de restauration d'onglets (I2) s'applique.
**Ma proposition** : page TV dans un **navigateur kiosque séparé** (Fully Kiosk Browser, ou WebView
dans un petit launcher), et Edge réservé aux jeux. Pour vérifier que la TV est bien revenue à
l'écran d'attente, je prendrai **deux signaux** :
- la page TV envoie `tv:ready` par WebSocket ;
- `dumpsys` confirme que l'application kiosque est au premier plan.

### I4 « LIBRE » mélangé avec les états de session
LIBRE est un état de la **box**, pas de la session. Je modéliserai deux machines à états distinctes :
- **Box** : `HORS_LIGNE`, `LIBRE`, `OCCUPEE`, `MANETTE_INDISPONIBLE`, `MAINTENANCE` ;
- **Session** : `RESERVEE` → `PAYEE` → `LANCEMENT` → `EN_JEU` → `FIN_PROCHE` → `TERMINEE`, plus
  `EXPIREE`, `ANNULEE` et `ERREUR_REFUND`.

L'unicité de la session active sera garantie par un **index unique partiel SQLite** sur
`sessions(box_id) WHERE state IN (états actifs)`.

### I5 Réservation de 5 min contre checkout SumUp
Une page Hosted Checkout SumUp reste valable 30 min. Un client lent (3-D Secure, changement de carte)
peut donc payer après l'expiration de sa réservation.
**Mesure** : je fixerai `valid_until` du checkout à l'heure d'expiration de la réservation, pour que
SumUp refuse le paiement après coup. Le remboursement automatique en cas de paiement tardif reste
en place comme filet de sécurité.

### I6 Prolongation payée trop tard
Une prolongation proposée à T-2 min peut être confirmée après T=0, alors que la session est déjà
TERMINEE et la box en cours de nettoyage.
**Ma proposition** :
- si un checkout de prolongation est en cours à T=0, la session est maintenue jusqu'à
  `EXTENSION_GRACE_SECONDS` (60 s par défaut) ;
- passé ce délai, la session se termine et un paiement qui arrive ensuite est **remboursé
  automatiquement**.

### I7 Packs « 2 parties pour 3 € »
Sans compte client, une « 2e partie » plus tard n'a pas de support : il faudrait un code ou un
crédit. **Ma proposition** : une table `offers` avec libellé, prix en centimes et durée en minutes.
Un « pack » correspond alors à **une session plus longue** (par exemple 40 min pour 3 €). Des
crédits avec code à usage unique pourront être ajoutés plus tard si tu en as besoin.

### I8 « Jeu non lancé après timeout » → remboursement
D'après I1, le système ne détecte que les échecs techniques : agent hors ligne, ADB en échec, Edge
pas au premier plan. Il ne voit pas un jeu bloqué sur un écran d'erreur xCloud.
**Ma proposition** :
- un bouton **« Signaler un problème »** sur le téléphone pendant les 3 premières minutes, qui
  t'alerte ;
- un **remboursement manuel en un clic** dans le back-office.

Pas de remboursement automatique sur simple déclaration du client, pour éviter les abus.

### I9 Multi-bars : qui encaisse ?
- Un seul `merchant_code` SumUp (le tien) pour tous les bars → tu reverses une commission au bar.
- Ou un `merchant_code` par bar → chaque bar a son compte SumUp et sa clé API.

**Ma proposition** : un `merchant_code` optionnel par bar en base, avec repli sur celui du `.env`.
Cela couvre les deux cas.

### I10 Un compte Microsoft et un abonnement par box
xCloud n'autorise qu'un flux à la fois par compte. Chaque box supplémentaire demande donc un compte
dédié et un abonnement de plus. Le coût augmente avec le nombre de box (voir §3).

### I11 Navigation libre dans Edge avec la manette
Avec la manette, ou en quittant le jeu, un client peut ouvrir un onglet et aller sur
account.microsoft.com : changer le mot de passe, se déconnecter, modifier le gamertag ou envoyer
des messages. Le mode kiosque Android ne protège pas l'intérieur du navigateur.
Mesures à combiner :
- confidentialité Xbox au maximum (pas de messages, pas de chat, profil privé) ;
- validation en deux étapes sur le compte, pour qu'un changement de mot de passe demande un code ;
- surveillance : l'agent vérifie l'URL au premier plan si `dumpsys` l'expose, sinon capture d'écran
  à la demande depuis le back-office (`adb exec-out screencap`).

Je propose d'ajouter cette capture d'écran à l'Étape 7.

### I12 Mineurs et PEGI
Le cahier des charges demande un « avertissement discret ». Pour les jeux PEGI 16 et 18, je propose
une **case à cocher « Je confirme avoir l'âge requis »** avant le paiement. C'est déclaratif, mais
cela trace la démarche. Il faut aussi l'avertissement épilepsie et photosensibilité habituel des
jeux vidéo (voir la checklist légale).

---

## 3. Frais SumUp et marge sur 1 €

> Chiffres à **vérifier sur ton compte SumUp**. Le taux en ligne vient de sources tierces : la page
> officielle française affiche 1,75 % pour le paiement en personne mais n'indique pas clairement le
> taux en ligne.

- Paiement en ligne (liens et checkout en ligne) : **~2,5 %, sans frais fixe** annoncé en France.
  À confirmer pour le Hosted Checkout par API.
- Il existe un abonnement « Paiements Plus » à taux réduit, rentable seulement à gros volume.
- À vérifier : SumUp **garde-t-il les frais en cas de remboursement** ? Y a-t-il un **montant
  minimum** par transaction en ligne ?

| Pour 1 € TTC payé | Avec TVA 20 % | En franchise de TVA (micro) |
|---|---|---|
| Hors taxe | 0,833 € | 1,000 € |
| Frais SumUp (2,5 % de 1 €) | −0,025 € | −0,025 € |
| **Net par partie** | **≈ 0,81 €** | **≈ 0,975 €** |

Le pourcentage SumUp sur 1 € est faible. **La vraie charge est fixe**, par box et par mois
(estimations à vérifier) :

| Poste | Estimation par mois |
|---|---|
| Abonnement Game Pass avec cloud (Ultimate ≈ 27 €/mois en France après la hausse d'octobre 2025) | ≈ 27 € |
| VPS, partagé entre tous les bars | ≈ 5 € |
| Amortissement matériel (box ~50 €, Raspberry Pi ~60 €, manette ~60 € sur 24 mois) | ≈ 7 € |
| Électricité | ≈ 2 € |
| **Total** | **≈ 40 €** |

Il faut donc environ **50 parties par mois et par box** pour être à l'équilibre avant la part du bar.
Si le bar prend 30 %, il en faut environ 70, soit 2 à 3 parties par jour. C'est jouable, mais
chaque remboursement ou chaque soir de panne pèse directement sur ce seuil.

---

## 4. Choix techniques que je ferai (sauf avis contraire)

| Sujet | Choix | Pourquoi |
|---|---|---|
| Runtime | Node.js LTS (22 ou 24) | Stable, supporté par PM2 et systemd |
| SQLite | `better-sqlite3` | API synchrone, transactions simples, très fiable pour un seul serveur |
| WebSocket | `ws` | Léger, standard, pas besoin de Socket.IO |
| ADB | `adb` en ligne de commande (spawn) | adbkit n'est plus maintenu ; le binaire officiel est le plus fiable |
| Alertes | Telegram par défaut, e-mail (SMTP) en option | Gratuit, instantané, sur ton téléphone |
| Mot de passe admin | `argon2` (argon2id) | Recommandation actuelle |
| Sessions admin | Cookie `httpOnly`, `secure`, `SameSite=Strict` + jeton CSRF | Exigence §9 |
| Montants | Entiers en **centimes** partout | Pas d'erreurs d'arrondi |
| Heures | Stockées en UTC (ms epoch), affichées en Europe/Paris | Pas de problème de changement d'heure |

---

## 5. Questions pour toi (à répondre avec les résultats de l'Étape 0)

1. **Démarrage du timer (I1)** : OK pour « bouton C'est parti, ou au plus tard 90 s après
   l'ouverture du jeu » ?
2. **Navigateur de la page TV (I3)** : OK pour un navigateur kiosque séparé (Fully Kiosk Browser,
   licence Plus ≈ quelques euros par appareil) ?
3. **Packs (I7)** : OK pour « pack = session plus longue » ?
4. **Prolongation tardive (I6)** : OK pour 60 s de grâce ?
5. **Encaissement (I9)** : un seul compte SumUp (le tien) pour commencer ?
6. **Alertes** : Telegram ?
7. **Âge (I12)** : OK pour la case à cocher sur les jeux PEGI 16 et 18 ?
8. **Matériel (R2, R3)** : on reste sur la H96 Max, ou tu testes aussi un Fire TV en parallèle ?
