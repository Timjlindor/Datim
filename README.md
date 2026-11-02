# DATIM CONNECT

**État courant et travaux : [registre central](docs/TRAVAUX.md).** Les rapports antérieurs sont des preuves historiques.

DATIM CONNECT accompagne les résidents haïtiens dans la préparation et le suivi de leurs besoins financiers locaux et internationaux. Next.js / Expo / FastAPI / PostgreSQL ; bleu encre, ivoire, vert profond et cuivre. DAT est la devise principale de l’espace portefeuille, à prix libre ; 50 HTG reste une référence indicative uniquement. Chaque actif et chaque environnement gardent leur source d’autorité distincte.

**État : développement incomplet, aucune opération réelle activée.** La session web, les lectures de portefeuille et les déclarations de comptes sont raccordées à Auth/API, avec configuration Supabase requise. Les workflows financiers restent incomplets. Les simulateurs sont isolés. Ne pas utiliser avec de l’argent réel.

- `apps/web` : espace utilisateur à `/` (une route par espace : `/international`, `/portefeuille`, `/brokers`, `/professionnels`…), présentation à `/decouvrir`, connexion à `/connexion`, terminal à `/marches`, back-office à `/admin`. Design system : [docs/design/DESIGN-SYSTEM.md](docs/design/DESIGN-SYSTEM.md).
- `apps/mobile` : application Expo, onglets Accueil / Étranger / Portefeuille / Activité / Plus, sécurité hors ligne de base.
- `services/api` : FastAPI, identité Supabase et primitives comptables PostgreSQL.
- `services/worker` : orchestration durable avec baux, reprise par statut et incidents ; simulation opt-in uniquement.
- `packages/api-client` : client HTTP et types OpenAPI générés.
- `packages/design-tokens` : tokens sémantiques partagés web et mobile (thèmes clair « Ivoire » et sombre « Nuit »).
- `supabase/migrations` : journal immuable, réservations/captures, gouvernance et RLS du rôle API ; plusieurs tables métier restent partiellement réservées.
- `infra` : fichiers de conteneurisation de développement.

## Démarrage

```sh
npm ci
npm run dev # http://localhost:2500
# Autre terminal
python3 -m venv .venv
.venv/bin/pip install -r services/api/requirements.lock.txt
.venv/bin/python scripts/run-api.py
# Mobile
npm run start -w apps/mobile
```

API et worker nécessitent Supabase et DATABASE_URL pour les lectures authentifiées. Aucune configuration n’active les paiements réels.

## Validation

```sh
npm run typecheck
npm run build
# Toutes les suites : montants, marché simulé, terminal, puis migrations + API sur PostgreSQL jetable
npm run test
PYTHONPATH=services/api .venv/bin/pytest services/api/tests/test_api.py -q
# Base de test isolée préalablement migrée, avec auth.users et rôles Supabase
PYTHONPATH=services/api TEST_DATABASE_URL='postgresql://…' .venv/bin/pytest services/api/tests -q
.venv/bin/python scripts/export_openapi.py
```

Les tests PostgreSQL écrivent des données de test : jamais sur une base réelle. `npm run test` exige PostgreSQL installé localement (`initdb`, `pg_ctl`, `psql`) : `npm run test:postgres` crée le cluster jetable, joignable par socket Unix seulement, puis le supprime. Seul, `npm run test:api` saute sans échouer les tests qui demandent une base.

Voir [l’état par lot](docs/DELIVERY.md), [les partenaires](docs/PARTNERS.md), [l’exploitation](docs/OPERATIONS.md) et [les résultats de vérification](docs/VERIFICATION.md).

## Configuration web

Copier les variables serveur `SUPABASE_URL`, `SUPABASE_ANON_KEY` et `DATIM_API_URL` dans `apps/web/.env.local`. Ne jamais y mettre une clé Supabase service-role. La clé anon est destinée à Supabase Auth ; les opérations financières ne passent jamais par le client Supabase.

L’API utilise un login PostgreSQL dédié membre de `datim_api`, avec NOBYPASSRLS et sans propriété des tables. Les contextes `datim.user_id` et `datim.internal` sont définis dans la transaction après validation du token. Les rôles internes proviennent uniquement de `app_metadata.datim_role` contrôlé côté Supabase (`compliance` ou `finance_admin`), jamais de `user_metadata`.

Le web conserve accès et renouvellement dans des cookies HttpOnly, SameSite Strict et Secure en production. Le serveur renouvelle l’accès quand le cookie court expire, sans rejouer une opération financière. La déconnexion efface les cookies et demande une révocation globale ; un jeton déjà émis peut rester valable jusqu’à expiration. Inscription, connexion mot de passe/code email, récupération par code et TOTP sont codées ; elles exigent des boîtes joignables, un email configuré et une validation utilisateur. Aucun compte prévalidé n’est créé pour contourner la confirmation.

## Mobile et contrats

Expo utilise `EXPO_PUBLIC_SUPABASE_URL`, `EXPO_PUBLIC_SUPABASE_ANON_KEY` (clé anon uniquement) et `EXPO_PUBLIC_DATIM_API_URL`. Pour un appareil réel, l’adresse API doit être accessible depuis cet appareil ; `localhost` désigne le téléphone lui-même. Les secrets serveur ne doivent jamais porter le préfixe EXPO_PUBLIC_.

Les types `WalletBalance` et `PaymentRecord` proviennent du schéma OpenAPI généré dans `packages/api-client`, utilisé par le web et le mobile. Exécuter `npm run generate:client` après toute modification du contrat FastAPI.

## Nouveau lot : orchestration et rapprochement simulés

Voir [le moteur de paiement](docs/PAYMENT_ORCHESTRATION.md). Pour tester les migrations depuis zéro et les scénarios financiers simulés dans un cluster PostgreSQL local jetable :

```sh
.venv/bin/python scripts/verify-postgres.py
```

PostgreSQL doit être installé localement avec `initdb`, `pg_ctl` et `psql` accessibles. Le script ne contacte ni Supabase ni un partenaire financier et ne modifie pas une base existante. Aucun mode ne débloque l’exécution financière de production.

## Extension produit du 21 octobre 2026

Domaines commerçants, flotte, laboratoire DAT, profil, assistance et indexeur ajoutés. Les mouvements du laboratoire sont fictifs et n’émettent aucun DAT sur la chaîne. Décision du 21 octobre : DAT à prix libre, 50 HTG comme référence indicative uniquement.

- [Spécification](docs/product/SPECIFICATION.md)
- [Registre complet des fonctions et limites](docs/product/LIVRAISON.md)
- [Hypothèses économiques](docs/product/ECONOMIE.md)
- [Exploitation et sécurité](docs/product/EXPLOITATION.md)

L’application n’est pas activée pour de l’argent réel. Le registre indique aussi les travaux logiciels restant à réaliser ; il ne faut pas présenter l’ensemble du périmètre comme terminé.

## Marchés DAT intégrés

`http://localhost:2500/marches` : terminal DAT/USD et DAT/EUR avec Lightweight Charts, indicateurs, dessins et alertes sauvegardés, ordres au marché et limités, annulation et historique en **simulation isolée**. Aucune cotation réelle ni exécution sur une plateforme externe. Voir [le fonctionnement et ses limites](docs/market/TERMINAL-V2.md). Tests : `npm run test:market`.

## Reprise : Haïti ↔ international

L’accueil privilégie envoyer, recevoir, alimenter, retirer, comprendre et suivre. `/api/health` relaie la préparation de FastAPI ; `/health` indique sa vie et `/ready` vérifie configuration et accès SQL. Les circuits sont conservés dans `public.financial_corridors`, les dossiers et étapes dans `international_journeys` et `journey_events`. L’aller et le retour sont indépendants ; aucun corridor réel n’est activé. Une simulation de processus produit un reçu explicitement sans valeur financière, sans débit ni crédit.

Le projet Supabase DATIM possède les migrations ; les logins API et terminal restreints sont provisionnés avec TLS vérifié. Le terminal utilise désormais PostgreSQL. Les anciennes données SQLite restent conservées, sans import automatique. Le dossier alternatif `datim-database` n’est pas déployé.

- [Architecture, matrice et correspondances des schémas](docs/product/REPRISE-ARCHITECTURE.md)
- [Démarrage, vérifications et dépendances](docs/product/REPRISE-DEMARRAGE.md)
- [Rapport concret de reprise](docs/product/REPRISE-LIVRAISON.md)

Les adresses DATIM proposées ont été refusées par Supabase (`email_address_invalid`) pendant la demande de vérification normale ; aucune validation administrative de ces identités n’a été effectuée. Le jalon hébergé authentifié complet reste à prouver après résolution de l’email.
