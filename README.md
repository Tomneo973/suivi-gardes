# Suivi des gardes

Application interne de suivi des gardes de patients et de personnes détenues.
Un seul fichier HTML qui s'ouvre dans Microsoft Edge : aucune installation, aucun serveur, aucune connexion Internet.
Les données sont des fichiers JSON dans un dossier partagé du réseau (2 utilisateurs simultanés).

## Démarrage rapide

1. Copier `suivi-gardes.html` sur chaque poste et l'ouvrir dans Microsoft Edge.
2. Cliquer sur **Choisir le dossier partagé** (dossier de données, hors de ce dépôt).
3. Si le dossier est vide, accepter la création d'un espace de données vide.
4. Choisir son profil (Opérateur, Consultation, ou Administrateur avec mot de passe).

## Contenu du dépôt

- `suivi-gardes.html` : l'application complète (HTML, CSS, JavaScript, sans dépendance).
- `outils/prototype-phase1.html` : page de test de l'accès au dossier réseau, des conflits et du verrou d'écriture (à passer sur les deux postes avant la mise en production).
- `docs/DOCUMENTATION.md` : documentation complète (utilisation, administration, fichiers, sauvegardes, dépannage, partie technique, validation).

## Données : à ne jamais publier

Le dépôt contient uniquement le code. Les fichiers de données (`config.json`, `admin.json`, dossiers `AAAA/Type/Mois`, `sauvegardes/`) contiennent des informations sensibles et sont exclus par `.gitignore`. Ne choisissez jamais le dossier du dépôt comme dossier de données, et gardez ce dépôt **privé**.

## Fonctions principales

- Gardes en cours sur une timeline 00:00–24:00 (quadrillage 15 min), services Matin / Après-midi / Nuit, agents par service, piquet.
- Gardes à traiter, historique, tableau de bord avec courbes de gardes simultanées.
- Export PDF paysage des services d'une garde sur une période, avec total des heures prestées.
- Une surveillance = un fichier JSON, rangé dans `AAAA/Type/Mois/` et nommé `AAAA_Axx_Nom_Statut.json` (Actif, Fini, Traite). Copies datées, détection des conflits, journal d'audit.
- Pool d'agents, contrôle LTR, référence semi-automatique, liens entre surveillances.
