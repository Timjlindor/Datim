# IMPLEMENTATION-PLAN — DAT Chain et DATIM CONNECT

Mise à jour de reprise : 2026-10-22. État courant : [registre central](docs/TRAVAUX.md). Le [rapport de reprise](docs/product/REPRISE-LIVRAISON.md) est conservé comme preuve historique. Les mesures ci-dessous datées du 20/21 restent des résultats historiques, pas de nouveaux tests exécutés à chaque reprise.

## Rapport d'état (à tenir à jour à chaque étape)

Les colonnes reprennent les 5 catégories du cahier des charges :
- **Impl. :** réellement implémenté.
- **Testé :** testé par une exécution réelle.
- **Partenaire :** nécessite un partenaire.
- **Autorisation :** nécessite une autorisation.
- **Opérationnel :** fonctionne réellement.

| Composant | Impl. | Testé | Partenaire | Autorisation | Opérationnel |
|---|---|---|---|---|---|
| Calendrier d'émission (`emission/`) | Oui | Oui : 17 tests unitaires, sur le Mac (Go 1.27.3, darwin/arm64), le 2026-10-20 | Non | Non | Non (pas encore de réseau) |
| Nœud `datd` (fork de evmd v0.7.3) | Oui | Compilé sur le Mac le 2026-10-20 ; `go vet ./...` propre ; `datd emission at` conforme à l'offre on-chain | Non | Non | Non |
| Genèse DAT (`datd init`, `datd testnet`) | Oui | `validate-genesis` réussi le 2026-10-20 ; plafond, métadonnées DAT et ERC-20 natif présents | Non | Non | Non |
| Devnet local à 1 validateur | Oui | Exécuté le 2026-10-20 : 110 blocs en 221 s (2,01 s par bloc) | Non | Non | Devnet local uniquement |
| Vérification de l'offre | Oui | Oui : `make supply` → `OK`, **écart de 0 adat** à neuf hauteurs, sur trois devnets, y compris après un transfert Cosmos et un transfert EVM | Non | Non | Devnet local uniquement |
| Transactions (Cosmos et EVM) | Hérité d'evmd | Oui : `bank send` et `eth_sendRawTransaction` réussis le 2026-10-20 ; soldes `x/bank` et `eth_getBalance` identiques | Non | Non | Devnet local uniquement |
| Test d'intégration en processus (`tests/network`) | Hérité, réparé | Oui : réussi en 33 s le 2026-10-20 (démarre l'application, produit des blocs, interroge le JSON-RPC) | Non | Non | Non |
| Testnet public | Non | Non | Opérateurs de validateurs | Non | Non |
| DAT Wallet (web et mobile) | Pont web EIP-1193 local ; mobile autonome restant | Extension réelle non testée dans cette reprise | Non | Non | Local partiel |
| DAT Explorer | Non | Non | Non | Non | Non |
| DAT Pay | Non | Non | Commerçants, partenaires de règlement | Probable (services de paiement) | Non |
| DAT Exchange | Non | Non | Teneurs de marché, liquidité | **Oui** (prestataire d'actifs virtuels selon les pays) | Non |
| Conversion HTG/USD | Simulateurs DATIM | Partiel (voir docs/VERIFICATION.md) | MonCash, NatCash, banques | Oui (BRH à confirmer) | Non |

## Décisions ouvertes

| N° | Sujet | Proposition | Décideur |
|---|---|---|---|
| D-1 | Allocation de genèse (le PoS exclut un pré-minage nul) | Option A de TOKENOMICS : stake minimal verrouillé par validateur indépendant | Fondateur |
| D-2 | Temps de bloc | 6 s (au lieu de 7 min, valeur liée au proof of work) | Fondateur |
| D-3 | Décimales | 18 (`adat`), et non 6 (`udat`), à cause de la contrainte Cosmos EVM v0.7 | Validé techniquement, à confirmer |
| D-4 | Paramètres de staking et de slashing | À proposer avant le testnet | Équipe |
| D-5 | `community_tax` | 2 % (valeur par défaut) | Fondateur |
| D-6 | Organisation GitHub et chemin du module Go | `github.com/datim-connect/dat-chain` provisoire | Fondateur |
| D-7 | Entité juridique et juridictions d'exploitation | Analyse par un avocat | Fondateur |
| D-8 | Activer IBC dès le testnet ? | Désactivé tant que non nécessaire | Équipe |

## Contraintes connues

- **Budget de 500 000 HTG (environ 3 800 USD au 2026-10-20) :** il suffit pour le devnet et le testnet (quelques serveurs). Il ne couvre **ni** un audit de sécurité, **ni** l'analyse juridique multi-pays, **ni** la liquidité d'un marché.
- **Licences :** aucune autorisation réglementaire n'a été obtenue. La conception d'une blockchain publique n'autorise pas l'exploitation d'un exchange, d'un service de conversion ou d'un service de paiement.
- **Validateurs :** 10 nœuds tenus par une seule équipe ne constituent pas un réseau décentralisé.

## Étapes (instruction §20 du cahier des charges)

| Étape | Contenu | État |
|---|---|---|
| 1–3 | Analyse, architecture, protocole | Brouillons : `docs/dat/PROTOCOL.md`, `TOKENOMICS.md`, `ARCHITECTURE.md` |
| 4 | Squelette du monorepo | `blockchain/dat-chain` créé ; structure existante conservée (voir ARCHITECTURE §5) |
| 5 | DAT Chain et devnet | **Terminée** : compilé, devnet exécuté, émission et offre vérifiées (écart nul) |
| 5b | Tests de consensus multi-nœuds (4 validateurs locaux) | **Vérifié le 2026-10-21** : quatre processus, accord blocs/état aux hauteurs 5, 9 et 15 ; progression avec un nœud arrêté, arrêt sans quorum, reprise et rattrapage (`make test-network`) |
| 6 | DAT Wallet | Pont navigateur local présent ; portefeuille autonome et validation restent à faire |
| 7 | DAT Pay | À faire |
| 8 | Moteur DAT Exchange (simulation séparée) | Terminal simulé développé et testé, réel absent |
| 9–10 | Backend DATIM et schéma PostgreSQL | Existant partiel (voir README racine) ; projection chain_blocks présente ; vérification des commits et indexeur runtime à terminer |
| 11 | Applications mobiles | Existant partiel (`apps/mobile`) |
| 12 | DAT Explorer | À faire |
| 13–15 | Intégration, infrastructure, préparation du lancement | À faire |

## Vérification exécutée le 2026-10-20 (sur le Mac, Go 1.27.3, darwin/arm64)

Tout a été obtenu : `make deps`, `make test` (17 tests), `go vet ./...`, `make build`, devnet
(110 blocs à 2,01 s), `make supply` → `OK` avec un écart de **0 adat**.

Mesures principales :
- division par deux observée on-chain au bloc 101 (10 → 5 DAT), événement `mint` portant le
  calendrier, l'ère et la hauteur ;
- offre on-chain = soldes de genèse + émission cumulée, écart nul, y compris après un transfert
  Cosmos et un transfert EVM : **les frais ne sont pas détruits**, ils passent par le fee
  collector puis `x/distribution` (2 % au community pool) ;
- solde `eth_getBalance` identique au solde `x/bank`.

### Corrections apportées pendant cette vérification

| Problème trouvé | Portée | Correction |
|---|---|---|
| `datd init` n'écrivait pas la genèse DAT : `max_supply` à 0, aucune métadonnée de dénom, pas de précompilé ERC-20 natif (le script du devnet le masquait avec `jq`) | Bloquant pour tout nouveau réseau | `cmd/datd/cmd/init.go` enveloppe `init` et applique les états de genèse de `genesis.go` |
| `datd testnet` (prévu pour l'étape 5b) finançait les comptes en `atest` et annonçait les métadonnées « Cosmos EVM / ATOM » | Bloquant pour le test à 4 validateurs | Tout est libellé en `adat`, métadonnées DAT, genèse DAT appliquée |
| `blocks_per_year` restait à la valeur par défaut du SDK (6 311 520, soit 5 s) | Valeur lue par les explorateurs | 5 259 600 (6 s) en genèse ; 15 778 800 (2 s) sur le devnet |
| `annual_provisions` était calculé au rythme du réseau principal, même sur le devnet | Valeur informative fausse | Chaque calendrier porte son propre nombre de blocs par an |
| La décision monétaire (plafonnement, `max_supply` de gouvernance) vivait dans `mint.go`, non testé | Risque sur le code qui crée les DAT | Déplacée dans le paquet `emission/`, testé : 4 tests de plus (17 au total) |
| Le script du devnet corrigeait la genèse au lieu de la vérifier | Masquait les régressions | Il vérifie désormais 10 paramètres et s'arrête en cas d'écart |
| Le test d'intégration en processus (`tests/network`) ne pouvait pas démarrer : pool de ports jamais alimenté, puis genèse de `x/bank` reconstruite à vide (métadonnées du dénom effacées, `x/vm` en panique). Les deux défauts viennent d'`evmd` v0.7.3 | Aucun test Go ne démarrait l'application | Corrigés : le test réussit en 33 s (`make test-all`, étiquette `norace`) |
| 6 fichiers hérités non formatés (ordre des imports après le renommage du module) | Propreté | `gofmt` appliqué ; `make test-all` ajouté pour les suites héritées |

### Prochaines vérifications

1. Réseau à 4 validateurs vérifié le 2026-10-21 : voir `docs/dat/PHASE2.md`. Restent les partitions et comportements byzantins.
2. Mesurer l'invariant d'offre sous charge et avec des appels de contrats EVM (et non un simple transfert).
3. Fixer les paramètres de staking et de slashing (décision D-4) avant tout testnet public.

## Phase 1 — renforcement du cœur local (20 octobre 2026)

- `make test-core` : suite dédiée aux transferts EVM signés et aux refus de chaîne incorrecte, signature invalide, fonds insuffisants et rejeu ; vérification du nonce, du crédit exact et du chaînage CometBFT.
- Banc d’essai corrigé : mempool transmise au backend RPC, file d’insertion configurée, mode CometBFT `app`, identifiants DAT et fonds de départ en `adat`. L’ancien test de liveness ne sollicitait pas ces chemins d’envoi.
- Aucun changement du calendrier d’émission ni activation de conversion HTG. Référence produit : 1 DAT = 50 HTG ; les DAT locaux restent sans valeur.
- Guide : `docs/dat/PHASE1.md`. Les phases réseau multi-nœuds, portefeuille et marché restent distinctes.
