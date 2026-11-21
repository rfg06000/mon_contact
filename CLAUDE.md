# Génération de vCard (.vcf) à partir d'une carte de visite

Quand l'utilisateur envoie dans la conversation une photo de carte de visite (ou une
capture d'écran équivalente) et demande une fiche de contact, suis ce déroulé :

1. **Lecture** : lis l'image directement (vision), sans dépendre d'un outil OCR externe.
2. **Présentation avant correction** : affiche toujours les informations lues sous forme
   de liste/tableau *avant* de générer le fichier, en distinguant clairement ce qui vient
   de la carte de ce qui vient d'une recherche web, et ce qui reste incertain (ex. code
   postal non visible). Ne saute jamais cette étape.
3. **Enrichissement par recherche web** : complète les données manquantes avec une
   recherche web — adresse postale exacte avec numéro de voie, téléphone au format
   international (`+33...`), site web, email professionnel, horaires d'ouverture.
   Pour le SIREN/SIRET d'une société française, utilise en priorité l'API publique et
   gratuite `https://recherche-entreprises.api.gouv.fr` (pas de clé requise) avant une
   recherche web générique.
4. **Génération** : une fois les informations confirmées (ou si l'utilisateur dit de ne
   pas attendre), génère le fichier `.vcf` et envoie-le avec `SendUserFile` pour qu'il
   soit ouvert directement dans l'app Contacts de l'iPhone. Ne te contente jamais de
   coller le texte du vCard dans le chat sans fichier joint.

## Structure des données (vCard 3.0)

- **Personne physique** : `N:NOM;Prénom;;;` — nom de famille en MAJUSCULES, prénom en
  casse habituelle. Si aucun nom de famille n'est identifié, laisse-le vide :
  `N:;Prénom;;;`.
- **Société / établissement** (pas de nom de personne identifié) :
  - Champ `N` strictement vide : `N:;;;;`
  - Nom complet dans `ORG` et `FN`
  - Restaurants et commerces de bouche : préfixe automatiquement le nom par
    « Restaurant » si l'activité l'évoque (mots-clés : restaurant, cuisine, kitchen,
    bistro, brasserie, traiteur, pizzeria...) et que ce mot n'est pas déjà présent.

## Formatage technique

- `VERSION:3.0`
- Champ `NOTE` : centralise horaires, informations d'accès, SIREN/SIRET.
- Dans `NOTE`, remplace chaque retour à la ligne par la séquence littérale `\n`.
- Replie (fold) toute ligne dépassant 75 octets, par un retour chariot suivi d'une
  espace avant la suite (RFC 2425/2426).
- N'inclus jamais de lien vers un réseau social.

## Historique

Ce dépôt contenait auparavant une page `scan.html` faisant l'OCR et la saisie dans le
navigateur (sans enrichissement automatique, faute d'API web utilisable gratuitement et
sans backend depuis une page statique GitHub Pages). Elle a été retirée : la génération
de vCard se fait désormais directement en conversation avec Claude, qui lit l'image,
enrichit les données par recherche web, présente le résultat pour correction, puis
envoie le fichier `.vcf`.
