# LeZachh — Cyber · Hacking éthique · Tech

Site vitrine responsive avec boutique, formation, annonces, lien WhatsApp, mesure d’audience sur accord et espace d’administration protégé.

## Espace administration

Ouvrez `/admin/` après le déploiement et connectez-vous avec le compte défini dans les secrets Cloudflare. L’administration permet de :

- consulter les visites consenties avec un identifiant aléatoire, les pages vues, la date et le domaine référent ;
- ajouter, modifier, afficher et supprimer des produits, dont le prix des guides et de la formation ;
- publier, modifier, masquer et supprimer des annonces ;
- changer l’arrière-plan avec un chemin du site (`/assets/ma-photo.jpg`) ou une URL HTTPS publique.

La connexion utilise un mot de passe haché avec PBKDF2, une session signée qui expire après 8 heures, un cookie `HttpOnly`/`SameSite=Strict`, une limite de tentatives et un contrôle d’origine contre les requêtes intersites. Les changements de contenu sont envoyés en JSON et enregistrés avec des requêtes SQL paramétrées.

Le site ne conserve pas l’adresse IP du visiteur. Le bandeau de statistiques propose explicitement **Accepter** ou **Refuser** et le pied de page permet de modifier ce choix. Le refus supprime aussi l’identifiant de suivi et la fiche associée. Les fiches de visite sont purgées après 12 mois lors d’une nouvelle visite consentie.

## Mise en ligne (ordre à respecter)

1. Cloudflare → **Workers & Pages → D1** : créer la base `lezachh-admin`, copier son identifiant dans `wrangler.toml` (`database_id`).
2. Appliquer les migrations : `npx wrangler d1 migrations apply lezachh-admin --remote` (applique `0001` et `0002`).
3. Générer l'empreinte du mot de passe : `node scripts/hash-admin-password.mjs` (14 caractères minimum).
4. Dans le projet Pages → **Settings → Variables and Secrets**, ajouter `ADMIN_USERNAME`, `ADMIN_PASSWORD_HASH`, `ADMIN_SESSION_SECRET` (48 octets aléatoires).
5. Déployer **le dossier `public/`** :
   - avec Git ou `npx wrangler pages deploy public --project-name=lezachh` (les fonctions sont prises en compte), ou
   - en glisser-déposer : lancer d'abord `npm run build:worker` (crée `public/_worker.js`), puis déposer `public/`. Lier ensuite la base D1 dans **Settings → Bindings** (nom `DB`).
6. Ouvrir `/api/health` : tout doit être `true`. Sinon `/admin/` affiche la liste des points manquants.

## Développement local

Configurez D1 localement après avoir remplacé l’identifiant dans `wrangler.toml` :

```powershell
npx wrangler d1 migrations apply lezachh-admin --local
npx wrangler pages dev public
```

Pour activer la connexion locale, ajoutez `ADMIN_USERNAME`, `ADMIN_PASSWORD_HASH` et `ADMIN_SESSION_SECRET` dans `.dev.vars` (ce fichier est ignoré par Git). Utilisez un mot de passe de développement distinct de votre mot de passe de production.

## Contrôles de sécurité

Exécutez `npm run security-check` pour vérifier les protections contre l’injection SQL, le XSS stocké, le CSRF, les URL dangereuses, les requêtes trop volumineuses, les sessions non authentifiées et l’enregistrement d’adresses IP.

## Fichiers

- `index.html` : vitrine et styles ;
- `site-enhancements.js` : rendu sûr des contenus, arrière-plan et choix d’audience ;
- `admin/` : interface de gestion ;
- `functions/api/` : API Cloudflare Pages, authentification, contenu, statistiques et visites ;
- `functions/_lib/security.js` : validation des entrées, sessions signées et contrôle d’origine ;
- `migrations/0001_init.sql` : tables D1 et offres initiales ;
- `_headers` : CSP et en-têtes de sécurité ;
- `wrangler.toml` : projet Pages et liaison D1.

## Nouveautés v6

- CSS et JavaScript déplacés dans `styles.css` et `app.js` : la politique de sécurité (CSP) n’autorise plus aucun script ou style intégré.
- Administration : tableau de fréquentation (visiteurs, pages vues, courbe sur 14 jours, pages et provenances principales), import de la photo d’arrière-plan depuis l’appareil, confirmation avant suppression.
- Corrections : bandeau des statistiques masqué par la barre de navigation, compteur « Guides en boutique » inexact, métadonnées EXIF retirées de la photo, plafond anti-saturation des visites.
