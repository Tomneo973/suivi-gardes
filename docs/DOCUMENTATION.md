# Suivi des gardes — Documentation

Application interne de suivi des gardes de patients et de personnes détenues · Version 1.5 · 29 octobre 2026

## 1. Vue d'ensemble

**À quoi ça sert.** Planifier, suivre et analyser les gardes : qui garde quelle personne, sur quel service, à quelles heures, avec quels agents, et combien d'heures et de piquets cela représente.

**Principes.**
- Un seul fichier, `suivi-gardes.html`, qui s'ouvre dans Microsoft Edge. Aucune installation, aucun serveur, aucun droit administrateur, aucune connexion Internet.
- Les données sont de simples fichiers JSON dans un dossier partagé du réseau, commun aux deux postes (2 utilisateurs simultanés).
- Rien n'est enregistré sans validation de l'utilisateur, et chaque enregistrement est protégé contre l'écrasement des modifications de l'autre poste.
- Interface en français.

**Prérequis.** Windows, Microsoft Edge récent (il doit proposer l'accès direct aux dossiers), accès en lecture et écriture au dossier partagé.

**Vocabulaire.**
- **Garde** (ou surveillance) : la surveillance d'une personne (patient ou détenu). Elle porte une référence facultative de type A01, et la personne gardée sert de nom d'usage.
- **Service** : un créneau type de la journée (Matin, Après-midi, Nuit).
- **Affectation** : un agent placé sur un service pour un jour donné, avec ses horaires.
- **Piquet** : droit à la rémunération de piquet, accordé à un agent arrivé sur la surveillance en moins de 3 heures.
- **Pool** : la liste des agents suivis (matricule, nom, prénom, contrat, actif ou inactif), gérée par l'administrateur.
- **LTR** : les règles de durée du travail et de repos contrôlées pour les agents du pool.

## 2. Démarrage rapide

1. Copier `suivi-gardes.html` sur chaque poste.
2. L'ouvrir dans Edge, cliquer sur **Choisir le dossier partagé** et autoriser l'accès en modification.
3. Si le dossier est vide, accepter la création d'un espace de données vide : l'application crée `config.json` et les dossiers `AAAA/Detenu` et `AAAA/Patient` pour l'année en cours.
4. Saisir son nom dans **Utilisateur** (il figure dans le journal d'audit) et choisir son **Profil**.
5. Premier passage en **Administrateur** : l'application demande de créer le premier compte (identifiant et mot de passe de 8 caractères minimum).
6. Aux ouvertures suivantes : **Reprendre le dossier précédent**. Edge redemande l'autorisation à chaque nouvelle session, c'est normal.
7. Dans **Administration**, vérifier les types de gardes (couleurs, ordre), saisir ou importer le pool d'agents et contrôler les règles LTR.

La zone d'en-tête affiche en permanence l'état de la connexion, l'heure de la dernière synchronisation, la dernière sauvegarde, les erreurs ou conflits éventuels, ainsi que l'heure et la date en grand à droite.

## 3. Profils et droits

- **Administrateur** (connexion obligatoire) : tous les droits, dont la console d'administration (pool d'agents, règles LTR, affichage, types), la modification et la suppression des gardes traitées, et leur réouverture avec justification, et gère la liste des cadres de piquet.
- **Opérateur** : crée et modifie les gardes en cours et à traiter, les lie entre elles, affecte les agents (garde en cours uniquement), clôture, traite, supprime une garde non traitée, exporte en PDF, renseigne le planning des piquets d'encadrement.
- **Consultation** : lecture seule. La garde apparaît sous sa référence (pas de nom), les risques et la provenance sont masqués, pas de recherche par nom, pas d'export PDF.

Le nom saisi dans « Utilisateur » est déclaratif : ce n'est pas une authentification. Seul le profil Administrateur est protégé par mot de passe.

## 4. Utilisation au quotidien

### 4.1 Cycle de vie d'une garde

**En cours → Terminée, à traiter → Traitée.**
- **Création** : bouton « + Nouvelle garde » dans « Gardes en cours ».
- **Clôture** (bouton Clôturer de la fiche) : la garde passe en « à traiter ». Les services en cours reçoivent comme fin réelle l'heure de la clôture, et les affectations futures sont retirées. Plus aucun agent ne peut être ajouté.
- **Traitement** (bouton Marquer traitée) : l'application vérifie que la personne, le type et le début sont renseignés et que les horaires sont cohérents, puis enregistre qui a traité la garde et quand.
- **Réouverture** : réservée à l'administrateur, avec une justification obligatoire. La garde repasse en « à traiter ».

La clôture n'est jamais automatique : dépasser l'heure de fin prévue ne clôture pas une garde.

### 4.2 Fiche d'une garde

Champs : référence (facultative), personne gardée, type, localisation, provenance (prison, pour un détenu), téléphone de service, radio de service, risques, début de la garde.

La **date de début** est la donnée principale. La **fin** n'est pas saisie : elle se déduit des agents planifiés (fin prévue = fin de la dernière affectation). Tant qu'aucun agent n'est affecté, une fin prévue facultative peut être saisie.

Modifier le type ou le mois de début d'une garde la déplace automatiquement dans le bon dossier.

**Référence (ID).** Elle est proposée automatiquement : la lettre combine le type et le mois (A = patient en janvier, B = détenu en janvier, C = patient en février, D = détenu en février, et ainsi de suite jusqu'à W et X en décembre), suivie d'un numéro de 01 à 99 (A01, puis A02). Elle se met à jour quand on change le type ou la date de début, et reste modifiable. Elle est facultative : un transfert ambulance, par exemple, n'a pas de référence et s'identifie par la personne. Seuls les types dont le nom contient « patient » ou « détenu » reçoivent une proposition ; un doublon déclenche un avertissement.

**Surveillances liées.** La fiche permet de relier une surveillance à une autre, par exemple urgences puis étage, ou une surveillance suivie d'un transfert ambulance. Le bouton **+ Lier à une garde** relie la garde à une garde existante, saisie par sa référence ou le nom de la personne. Le bouton **+ Créer la suite** crée une nouvelle garde déjà liée, préremplie avec la personne, le type, la provenance et les risques, qui commence à la fin de la garde actuelle. Le lien fonctionne dans les deux sens, indique « avant » ou « après », s'ouvre d'un clic et se retire avec ✕. Les gardes liées portent une icône de chaîne dans le Gantt et les tableaux. Si le mois d'une garde liée n'est pas chargé, seule sa référence s'affiche.

### 4.3 Services et agents

Horaires fixes de chaque jour :
- **Matin** : 07h45 – 16h00
- **Après-midi** : 15h45 – 24h00
- **Nuit** : 23h45 – 08h00 (le lendemain)

Bouton **+ Agents** dans la fiche :
- choisir le service ;
- saisir un ou plusieurs agents séparés par une virgule : nom, prénom ou matricule pour un agent du pool, ou un nom libre pour un agent hors pool ;
- choisir le jour, ou une période « Du… Au… » : une affectation est créée par jour et par agent (62 jours au maximum) ;
- heures de début et de fin facultatives pour remplacer les horaires du service ;
- case piquet (voir 4.4).

Un agent peut donc faire le matin le lundi et un autre agent le matin le mardi. Les boutons ✎ et ✕ de la fiche modifient ou retirent une affectation (le retrait n'est possible que sur une garde en cours).

L'application signale les horaires incohérents et les chevauchements d'un même agent, y compris entre deux gardes, puis demande confirmation. Matin et Après-midi se chevauchent de 15 minutes par conception : un même agent placé sur les deux le même jour déclenche donc un avertissement.

**Agents du pool.** Un agent du pool est retrouvé par son matricule, par son nom et prénom (dans les deux ordres) ou par son seul nom s'il est unique. Il est alors toujours écrit de la même façon (par exemple DUPONT Jean) et son matricule est enregistré dans l'affectation. Si plusieurs agents du pool correspondent à ce qui est saisi (par exemple « Dupont » pour DUPONT Marc, matricule 12345, et DUPONT Thomas, matricule 54321), une fenêtre s'ouvre avec une liste déroulante pour choisir la bonne personne, affichée avec son nom, son prénom et son matricule. Annuler ce choix annule l'enregistrement. La même fenêtre s'ouvre quand on modifie l'agent d'une affectation. Un agent hors pool reste saisissable librement, mais il n'est ni suivi ni contrôlé. Planifier un agent inactif déclenche une alerte.

**Contrôle LTR.** Pour les agents du pool, l'application vérifie à l'entrée en planification :

- 6 jours consécutifs au maximum (un jour compte à sa date de début de service), suivis de 35 heures de repos avant le service suivant ;
- 10 heures de repos minimum entre deux services ;
- pas plus de 13 heures de travail dans une plage de 14 heures en journée (jour = 06:00–23:00) ;
- pas plus de 11 heures de travail dans une plage de 13 heures qui touche la nuit (nuit = 23:00–06:00). Le travail compte sans double décompte quand deux services se chevauchent.

En cas d'infraction, une alerte détaille les règles enfreintes et l'opérateur choisit de confirmer ou d'annuler ; une confirmation est inscrite dans le journal d'audit. Les infractions restent signalées par ⚠ dans le Gantt et dans la rubrique « À vérifier » de la fiche. Les seuils se règlent dans l'Administration. Le contrôle ne tient compte que des mois chargés.

### 4.4 Piquet

Chaque affectation porte une case **Droit à la rémunération de piquet**, décochée par défaut. On la coche quand l'agent est arrivé sur la surveillance en moins de 3 heures.
- Elle se coche d'un clic directement dans la fiche, à côté du nom de l'agent, ou dans le formulaire d'affectation.
- Chaque changement est inscrit dans le journal d'audit.
- Le tableau de bord totalise les piquets par type de garde (voir 4.7).
- Un agent avec droit au piquet est suivi de « (piquet) » dans le Gantt.

Quand on ajoute plusieurs agents d'un coup, la case s'applique à tous : pour un seul agent, la cocher ensuite dans la fiche.

### 4.5 Gardes en cours : la timeline

L'écran affiche les gardes en cours sur une échelle de 00:00 à 24:00, pour le jour choisi (bouton « Aujourd'hui » pour revenir au jour courant).
- **Quadrillage** : trait marqué à chaque heure, trait léger tous les 15 minutes. Une ligne rouge indique l'heure actuelle (actualisée toutes les 30 secondes).
- **Barre principale** : de la couleur du type de garde, avec sur une seule ligne : lieu, téléphone, radio, date de début, risques, agent actuel, prochain agent. Si la barre est trop étroite (moins de 55 % de la journée), la ligne s'affiche dans une pastille de même couleur posée à côté.
- **Clic sur une garde** : déplie ses services, une ligne par service dans l'ordre Matin, Après-midi, Nuit. Les barres sont de la couleur du service et portent le nom des agents et leurs horaires. Les agents aux horaires identiques partagent une barre ; un chevauchement partiel agrandit la ligne. Un service sans agent est affiché avec « aucun agent », ce qui permet de repérer un trou de planning.
- **Tri** : par type de garde (ordre défini dans l'administration, par exemple détenus puis patients), par début ou par référence.
- Les gardes à cheval sur minuit sont découpées par jour (◂ ▸).

Un agent du pool en infraction LTR est suivi de ⚠ dans les barres de service ; un agent avec droit au piquet est suivi de « (piquet) ».

### 4.6 À traiter et Historique

- **À traiter** : liste triable des gardes terminées. Un clic ouvre la fiche pour vérifier, corriger puis marquer traitée.
- **Historique** : recherche combinable par référence ou ID, type, localisation, dates, agent et statut. Un clic ouvre la fiche (lecture seule pour une garde traitée, sauf administrateur).

### 4.7 Tableau de bord

Filtres : période, type de garde, localisation. Indicateurs : nombre de gardes, en cours, à traiter, traitées, durée cumulée, heures-agents, piquets, agents distincts, services. Tableaux : gardes par type, piquets par type de garde (nombre et heures), gardes par jour, mois ou année. Deux courbes : le nombre de gardes simultanées par type sur une journée (pas de 15 minutes) et le pic quotidien sur un mois.

### 4.8 Export PDF

Bouton **Exporter en PDF** dans la fiche (Opérateur et Administrateur), puis fenêtre de paramètres : jour de début, jour de fin, horaires prévus et réels ou prévus seulement.
- Le PDF est en A4 paysage, trié chronologiquement : date, service, agent, horaires prévus, horaires réels, durée, piquet.
- Il ne dépend pas du statut : une garde commencée un vendredi et encore en cours le lundi s'exporte pour le week-end. Un service qui déborde sur la période est inclus en entier (la nuit du vendredi au samedi apparaît dans un export du samedi).
- Il se termine par le **total des heures prestées sur l'ensemble de la garde**, suivi du nombre d'affectations et d'agents, du total de la période exportée et du nombre de piquets.
- Le fichier est téléchargé directement, sous la forme `services_ID_du_au.pdf`. Chaque export est inscrit dans le journal d'audit.

### 4.9 Supprimer une garde

Bouton **Supprimer** dans la fiche, avec confirmation (le profil Opérateur ne peut pas supprimer une garde traitée). L'événement est inscrit dans le journal d'audit de `config.json`, les liens avec d'autres gardes sont retirés, et une dernière copie du fichier est conservée dans `sauvegardes/` (nom terminé par `_supprimee`).

### 4.10 Panneau latéral

La fiche s'affiche dans un panneau à droite. La languette au bord droit de l'écran le replie pour ne garder que les timelines ; son état est mémorisé. Cliquer une garde dans un tableau le rouvre automatiquement.

### 4.11 Horloge et affichage

L'heure (avec les secondes) et la date s'affichent en grand en haut à droite de la fenêtre, et une petite icône aux trois barres de Gantt sert de favicon. Les tailles de police et des éléments se règlent dans l'Administration (voir la section 5).

### 4.12 Export PDF des agents sur une période

Bouton **Exporter les agents (PDF)** en haut de la fenêtre (Opérateur et Administrateur). Une fenêtre demande le jour de début et le jour de fin, le tri (chronologique, par numéro de garde ou par agent) et les horaires (prévus et réels, ou prévus seulement).

Le PDF, en A4 paysage, liste toutes les affectations de toutes les gardes qui chevauchent la période, quel que soit le statut des gardes : numéro de garde, personne surveillée, agent, matricule, date, service, horaire prévu, horaire réel et piquet. Il se termine par le total des heures prestées. Il sert à planifier les agents dans le logiciel de planification sans attendre les retours de carnet de service, par exemple pour toutes les gardes du vendredi au lundi. L'application charge au besoin les mois de la période et le mois précédent. Tous les types de service sont pris en compte, y compris « Pause ». Tris : chronologique (début de chaque service, puis numéro de garde quand plusieurs services commencent en même temps) ; par numéro de garde (tous les services d'une garde dans l'ordre chronologique sur la période, puis la garde suivante) ; par agent. Le fichier est téléchargé sous la forme `agents_du_au.pdf` et l'export est inscrit dans le journal d'audit.

### 4.13 Piquets d'encadrement

Les piquets d'encadrement sont les cadres que l'on peut joindre en dehors des horaires de bureau. Ils remplacent le fichier Excel (une croix par jour et par cadre). Trois pages y sont consacrées.

- **Piquet du jour** : le cadre de piquet aujourd'hui, avec son numéro de téléphone, les sept prochains jours et, s'il y en a, les jours du mois en cours sans piquet. Visible pour tous les profils.
- **Planning piquets** : une grille cadres par jours pour un mois (flèches et sélecteur de mois). Un clic sur une case attribue le piquet d'un jour à un cadre, un second clic le retire ; un jour n'a qu'un seul cadre. Le bouton **Attribuer une période** attribue un cadre sur plusieurs jours, en conservant ou en remplaçant les jours déjà attribués. Modifiable par les profils Opérateur et Administrateur, en lecture seule pour Consultation.
- **Cadres de piquet** : la liste restreinte des cadres qui peuvent être de piquet (nom, prénom, téléphone, actif ou inactif). Seuls les cadres actifs peuvent être placés de piquet. Modifiable par l'administrateur uniquement.

Un piquet doit être défini pour chaque jour. Dès qu'un jour du mois en cours n'a pas de piquet, un bandeau d'alerte s'affiche sous le menu, avec les jours concernés, et un compteur apparaît sur l'onglet « Piquet du jour ». Chaque changement est inscrit dans le journal d'audit. Les données sont dans `cadres.json` et dans un fichier par mois (`Piquets/AAAA/Mois.json`).

### 4.14 Disponibilités théoriques et capacités

**Disponibilités.** Dans l'Administration, le bouton **Dispo.** d'un agent du pool ouvre une grille : les 7 jours de la semaine en colonnes, les services en lignes (Matin, Après-midi, Nuit et les services ajoutés, sauf « Pause »). On coche les créneaux où l'agent est disponible en théorie ; « Tous les jours » coche une ligne, « Tout cocher » et « Tout décocher » agissent sur toute la grille. La colonne « Disponibilités » du pool indique le nombre de créneaux cochés. Les disponibilités sont enregistrées dans `agents.json` et réservées à l'administrateur.

**Page Capacités** (profils Opérateur et Administrateur). Pour la semaine choisie (flèches, sélecteur de date, « Cette semaine »), un tableau croise les services et les 7 jours. Chaque case affiche **agents planifiés / agents disponibles en théorie**, calculés sur les agents actifs du pool : vert quand il reste de la marge, orange quand tous les agents disponibles sont planifiés, rouge quand il y a plus d'agents planifiés que de disponibles, gris quand rien n'est prévu. Une ligne de total donne le cumul des créneaux de chaque jour. Un clic sur une case liste les agents disponibles (planifiés ou non) et les agents planifiés hors disponibilités déclarées, y compris les agents hors pool. Un service est compté à la date de son début : la nuit de 23:45 compte pour le jour où elle commence. L'application charge au besoin les mois de la semaine.

## 5. Administration

Accessible uniquement au profil Administrateur après connexion.
- **Types de gardes** : couleur modifiable, ordre modifiable avec les flèches ↑ ↓ (il détermine le tri), ajout d'un type. Un nouveau type crée son dossier `AAAA/Type` pour l'année en cours.
- **Types de services** : couleur modifiable, ajout d'un service avec ses horaires facultatifs.
- **Pool d'agents** (enregistré dans `agents.json`) : liste avec recherche (matricule, nom, prénom, contrat, statut), ajout, modification, activation ou désactivation, suppression (les plannings déjà saisis gardent le nom et le matricule de l'agent), saisie de ses disponibilités théoriques et import d'une liste collée (une ligne par agent : matricule ; nom ; prénom ; contrat). Pour chaque agent, le tableau affiche les services, les heures, les piquets et les alertes LTR sur les mois chargés. Renommer un agent ne modifie pas les plannings déjà saisis.
- **Contrôle LTR** : les sept seuils (jours consécutifs, repos minimum, repos après la série maximale de jours, travail maximum et plage de jour, travail maximum et plage de nuit).
- **Affichage** : six réglages appliqués à tous les postes (police générale, hauteur des barres de garde, hauteur des barres de service, largeur de la colonne des libellés, largeur du panneau latéral, taille de l'horloge), avec un bouton pour rétablir les valeurs par défaut.
- **Comptes** : changer son mot de passe, ajouter un compte administrateur.
- **Journal d'audit** : les 100 derniers événements des mois chargés (qui, quand, quelle action, quel objet, détails).

## 6. Règles de calcul

- **Rattachement à une période** : une garde est comptée à sa date de début.
- **Durée d'une garde** : fin réelle (sinon prévue) moins début.
- **Heures-agents** : somme des durées des affectations, horaires réels si les deux sont renseignés, sinon horaires prévus. Les services nommés « Pause » sont exclus. Deux agents pendant quatre heures font huit heures-agents.
- **Piquets** : nombre d'affectations dont la case piquet est cochée, avec leurs heures, regroupé par type de garde.
- **Agents distincts** : un agent n'est compté qu'une fois, par matricule pour un agent du pool et par nom sinon, sans distinction de majuscules.
- **Services** : nombre de couples distincts (service, jour).
- **Dates et heures** : stockées au format ISO en UTC et affichées en heure locale.

## 7. Fichiers, enregistrement et sauvegardes

### 7.1 Organisation du dossier partagé

- `config.json` : types de gardes (nom, couleur, dossier), types de services, règles LTR, réglages d'affichage, et journal des changements de configuration et des suppressions de gardes.
- `agents.json` : le pool d'agents (matricule, nom, prénom, contrat, actif ou inactif) et le journal de ses modifications. Au premier démarrage avec cette version, un pool saisi auparavant dans `config.json` est repris automatiquement dans ce fichier.
- `admin.json` : comptes administrateur. Les mots de passe y sont hachés (PBKDF2-SHA256, sel aléatoire), jamais en clair.
- `AAAA/Type/Mois/` : une surveillance = un fichier JSON, par exemple `2026/Patient/Janvier/2026_A01_Dupont_Actif.json`. Le mois est écrit en français (Janvier à Décembre). Une garde reste dans le mois de sa date de début. Les dossiers de type s'appellent `Patient`, `Detenu`, ou le nom d'un type créé dans l'administration.
- Nom du fichier : `AAAA_Référence_Nom_Statut.json`, ou `AAAA_Nom_Statut.json` sans référence. Le statut vaut `Actif` (garde en cours), `Fini` (garde clôturée mais pas traitée) ou `Traite` (garde finie et traitée). Le fichier est renommé automatiquement à chaque changement de statut, de référence ou de nom ; si deux fichiers auraient le même nom, un suffixe de 4 caractères est ajouté.
- `sauvegardes/` : copies datées, une par fichier enregistré, 300 conservées.
- `cadres.json` : la liste des cadres pouvant être de piquet d'encadrement (nom, prénom, téléphone, actif ou inactif) et le journal de ses modifications.
- `Piquets/AAAA/Mois.json` : le planning des piquets d'encadrement, un fichier par mois (par exemple `Piquets/2026/Octobre.json`), avec un cadre par jour.
- `gardes.verrou` : verrou temporaire d'écriture, présent seulement pendant un enregistrement.

### 7.2 Comment se passe un enregistrement

1. Prise du verrou (il expire de lui-même après 8 secondes).
2. Relecture sur le disque des fichiers concernés : la garde modifiée, ses gardes liées le cas échéant, ou `config.json`.
3. Application de la modification sur ces données fraîches.
4. Validation (JSON, version de schéma, identifiants uniques).
5. Écriture au nom et à l'emplacement calculés (statut, référence, nom, mois de début), puis relecture de contrôle.
6. Suppression de l'ancien fichier en cas de renommage ou de déplacement, copies datées dans `sauvegardes/`, puis libération du verrou.

L'en-tête indique chaque étape : en cours, confirmé, échoué ou conflit. Les écritures des deux postes sont ainsi mises à la file, et une erreur ne produit jamais de confirmation erronée.

### 7.3 Conflits

Chaque garde porte un numéro de version. Si l'autre poste l'a modifiée depuis l'ouverture de votre formulaire, l'enregistrement est refusé avec un message de conflit, les données les plus récentes sont rechargées, et votre saisie reste dans le formulaire pour être reprise. Il en va de même si l'autre poste a renommé ou déplacé le fichier entre-temps, par exemple en clôturant la garde : le fichier que vous aviez n'existe plus sous ce nom. L'application ne remplace jamais silencieusement les modifications d'un autre utilisateur. Elle vérifie aussi toutes les 5 secondes si les fichiers ont changé et met l'écran à jour.

### 7.4 Chargement par mois

Pour rester rapide, l'application charge le mois courant et le mois précédent (dossiers `AAAA/Type/Mois`). Les autres mois se chargent à la demande, par un filtre de dates (tableau de bord, historique, jour du Gantt) ou avec le bouton **Tout charger**. Une garde ancienne encore en cours n'apparaît donc que si son mois est chargé. Toutes les 5 secondes, l'application relit la liste des fichiers des mois chargés et les gardes non traitées ; les gardes traitées sont relues toutes les 30 secondes.

### 7.5 Restaurer une sauvegarde

1. Fermer l'application sur les deux postes.
2. Dans `sauvegardes/`, repérer la copie voulue. Son nom est `horodatage_AAAA_Type_Mois_NomDuFichier.json`, par exemple `2026-10-18T19-50-12-345Z_2026_Patient_Octobre_2026_A01_Dupont_Actif.json`. Pour la configuration, le nom se termine par `_config.json` ; pour le pool d'agents, par `_agents.json` ; pour les cadres, par `_cadres.json` ; pour un planning de piquets, par `Piquets_AAAA_Mois.json` ; pour une garde supprimée, par `_supprimee.json`.
3. La copier dans `AAAA/Type/Mois/` (ici `2026/Patient/Octobre/`) sous son nom d'origine (`2026_A01_Dupont_Actif.json`), après avoir supprimé de ce dossier la version plus récente de la même garde si elle existe. Le statut inscrit dans le nom doit correspondre au contenu du fichier.
4. Vérifier que le fichier s'ouvre dans un éditeur de texte et que son contenu est du JSON valide, puis rouvrir l'application.

## 8. Sécurité et limites

- **Protection des fichiers** : la vraie barrière est la configuration des droits Windows du dossier partagé. Il faut en limiter l'accès aux personnes autorisées, et appliquer le même niveau de protection au dossier `sauvegardes/`. Quelqu'un qui accède aux fichiers peut les modifier hors de l'application.
- **Mot de passe administrateur** : il est vérifié dans le navigateur. C'est un garde-fou, pas une sécurité forte. Si le mot de passe est perdu, une personne ayant accès au dossier peut supprimer `admin.json` pour recréer un compte.
- **Données sensibles** : aucune donnée ne quitte le poste et le réseau interne, il n'y a ni télémétrie ni service externe. Le profil Consultation ne voit ni les noms, ni les risques, ni la provenance.
- **Verrou d'écriture** : il repose sur un fichier et n'est pas une garantie transactionnelle. Il doit être validé par un test à deux postes dans l'environnement réel (voir section 11). Si aucune écriture fiable n'est possible, il faut limiter le mode multi-utilisateur ou séquencer les enregistrements.
- **Changements d'heure** : les durées sont calculées sur les dates réelles, mais la nuit du changement d'heure n'a pas de traitement particulier.
- **PDF** : polices standard avec les accents d'Europe occidentale ; un texte trop long est coupé par « … ».

## 9. Dépannage

- **« Ce navigateur ne permet pas l'accès direct aux fichiers »** : utiliser Microsoft Edge récent, avec le fichier ouvert en local. Une politique de sécurité de l'organisation peut aussi bloquer cette fonction.
- **Edge redemande l'autorisation** : normal à chaque nouvelle session, cliquer sur « Reprendre le dossier précédent ».
- **« Fichiers verrouillés par un autre poste »** : attendre quelques secondes. Si le message persiste alors que personne n'enregistre, supprimer `gardes.verrou`.
- **« Conflit détecté »** : relire les données rechargées, puis refaire la modification (la saisie est conservée dans le formulaire).
- **« Fichier illisible » ou « JSON invalide »** : l'application bloque toute écriture. Restaurer la dernière copie saine (7.5).
- **Une garde est introuvable** : son mois n'est pas chargé. Utiliser un filtre de dates ou « Tout charger ».
- **« Aucun service planifié sur cette période »** : le PDF n'a aucune affectation entre les deux jours choisis. Élargir la période.
- **Échec de sauvegarde dans l'en-tête** : l'enregistrement a réussi, mais la copie datée a échoué. Vérifier l'espace disque et les droits sur `sauvegardes/`.
- **Fenêtre « Plusieurs agents correspondent »** : choisir la bonne personne dans la liste. Si deux agents ont exactement les mêmes nom et prénom, le matricule les distingue dans la liste.
- **Pas d'alerte LTR pour un agent** : il est hors pool (nom libre), ou son planning d'un autre mois n'est pas chargé.
- **Le nom d'un fichier de garde ne correspond plus à son statut** (renommage manuel) : l'application se fie au contenu du fichier et le renomme au prochain enregistrement. Éviter de renommer ou de déplacer les fichiers à la main.

## 10. Pour le développeur

### 10.1 Modèle de données

Version de schéma : 1 (vérifiée à chaque lecture).

- **`config.json`** : `types` (liste de nom, couleur, dossier ; l'ordre de la liste est l'ordre de tri), `services` (nom, couleur, début, fin ; la fin peut valoir `24:00`), `ltr` (`jours`, `repos`, `sem`, `jour`, `pjour`, `nuit`, `pnuit` ; facultatif), `affichage` (facultatif), `audit`, `rev`.
- **`agents.json`** : `pool` (`mat`, `nom`, `prenom`, `contrat`, `actif`, `dispo` : jours 1 à 7 du lundi au dimanche, avec la liste des services), `audit`, `rev`.
- **`cadres.json`** : `cadres` (`id`, `nom`, `prenom`, `tel`, `actif`), `audit`, `rev`.
- **Fichier de piquets** (`Piquets/AAAA/Mois.json`) : `mois`, `jours` (clé `AAAA-MM-JJ`, valeur : `id` et `nom` du cadre), `audit`, `rev`.
- **Fichier de garde** (un par surveillance) : la garde elle-même, `schema_version` et son journal `audit`.
- **Garde** : `id`, `numero` (la référence, éventuellement vide), `personne`, `type`, `localisation`, `provenance`, `tel_service`, `radio_service`, `risques`, `statut` (`en_cours`, `a_traiter`, `traitee`), `debut_prevu`, `debut_reel`, `fin_prevue`, `fin_reel`, `cree_le`, `cloture_le`, `traite_le`, `traite_par`, `v` (version de la garde), `liens` et `services`.
- **Lien** : `id` de la garde liée et `numero` (repère de secours si cette garde n'est pas chargée).
- **Service** : `id`, `type`, `aff` (liste d'affectations).
- **Affectation** : `id`, `agent` (forme canonique pour un agent du pool), `mat` (matricule, vide hors pool), `piquet` (booléen), `debut`, `fin`, `debut_reel`, `fin_reel`.
- **Événement d'audit** : `id`, `le`, `user`, `role`, `action`, `objet`, `details`.
- **`admin.json`** : `comptes` (id, sel, hash).

### 10.2 Organisation du code

Tout est dans `suivi-gardes.html` (HTML, CSS, JavaScript, sans bibliothèque ni CDN). Le script est découpé en blocs :
- état et utilitaires (dates, couleurs, droits, pool d'agents, contrôle LTR, références) ;
- **accès aux fichiers**, isolé dans un seul bloc : verrou, lecture et écriture d'un fichier par garde (nom, dossier, renommage), validation, copies datées, chargement par dossier, connexion administrateur. C'est le seul endroit à remplacer si Edge ou le réseau imposait une autre solution ;
- formulaires et actions (une fonction par bouton) ;
- génération du PDF, écrite à la main sans bibliothèque ;
- vues : tableau de bord, gardes en cours, à traiter, historique, administration, fiche ;
- actualisation périodique et horloge.

Toute modification de données passe par la fonction d'enregistrement unique, qui garantit verrou, validation, relecture et copie datée.

### 10.3 Modifications courantes

- **Changer les horaires d'un service** : éditer `services` dans `config.json`, avec l'application fermée sur les deux postes. Seules les nouvelles affectations suivent ; les existantes gardent leurs horaires.
- **Ajouter un type de garde ou de service, un agent du pool, régler le LTR ou l'affichage** : par l'administration (jamais à la main).
- **Régler la conservation des sauvegardes ou la durée du verrou** : constantes dans le bloc d'accès aux fichiers (300 copies, 8 secondes).
- **Changer le format du nom de fichier ou l'arborescence** : fonction `gPath` du bloc d'accès aux fichiers ; prévoir la migration des fichiers déjà écrits.
- **Migration de schéma** : incrémenter `schema_version` et prévoir la conversion à la lecture.

## 11. Validation avant mise en production

Les tests automatiques réalisés pendant le développement ont utilisé un dossier simulé en mémoire : création, affectations, conflit de version, clôture, traitement, suppression, comptes administrateur, piquets, pool d'agents, contrôle LTR, références, liens entre gardes, PDF, et renommage ou déplacement des fichiers selon le statut et le mois. Ils n'ont pas été exécutés dans Edge sur le lecteur réseau réel. À valider sur place :

- Edge peut lire et écrire le dossier partagé depuis les deux postes, et l'autorisation se redonne après fermeture et réouverture.
- Le prototype `prototype-phase1.html` (test d'accès et de concurrence) passe sur les deux postes en même temps, en mode « avec verrou » : compteur final égal à la somme des écritures réussies des deux postes.
- Une modification de l'un est visible chez l'autre en quelques secondes.
- Deux modifications simultanées de la même garde produisent un conflit sans perte de saisie.
- Une garde clôturée sur un poste pendant qu'elle est ouverte sur l'autre produit un conflit sans perte de saisie.
- Un fichier JSON volontairement corrompu (sur une copie de test) est détecté et bloque l'écriture.
- La procédure de restauration a été répétée une fois.
- Aucune requête réseau externe n'apparaît dans les outils de développement d'Edge.
- Les droits Windows du dossier de données et de `sauvegardes/` sont limités aux personnes autorisées.
- Les noms de fichiers (accents, caractères spéciaux dans les noms de personnes) sont acceptés par le lecteur réseau et par les outils de sauvegarde de l'organisation.
- Les temps de chargement et de mise à jour sont acceptables avec les volumes réels (nombre de fichiers par mois) sur le lecteur réseau.
- Les règles LTR (seuils, définition des plages de jour et de nuit, décompte des jours consécutifs) sont validées avec les ressources humaines.
- L'informatique a validé le mode d'identification (nom déclaré, mot de passe administrateur) et la politique de conservation des données et des sauvegardes.
