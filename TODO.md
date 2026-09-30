# TODO

## Error Handling Enrichment | Add a specific error for insufficient XLM reserve
- [x] Step 1: Add `makeInsufficientXLMReserveError()` to `src/utils/errors.js`
- [x] Step 2: Update `src/middleware/errorHandler.js` to handle `isInsufficientXLMReserve`
- [x] Step 3: Add tests in `tests/errorHandler.test.js`
- [x] Step 4: Run tests to verify

## Response Normalisation | Normalise GET /account/:id/sequence response shape
- [ ] Analyse expected vs actual response shapes
- [ ] Implement normalization changes
- [ ] Run tests to verify

## Cache bypass documentation (?fresh=true)
- [ ] Confirm all endpoints that respect `?fresh=true` (likely `/network-status` and `/fee-estimate` and their subroutes).
- [ ] Update `README.md` with a "fresh cache bypass" section and request examples.

## Sanitize middleware: extend to req.body
- [ ] Update `src/middleware/sanitize.js` to sanitize `req.body` (strings, arrays, nested objects).
- [ ] Enforce the same max-length rule (500 chars) for body string values.
- [ ] Add/extend tests in `tests/sanitize.test.js` for body trimming, null-byte stripping, and 400 on >500 length.

## Standardize query parameter validation error messages (Option A)
- [ ] Update `src/utils/validators.js` error messages to use a single template (e.g., `Query parameter '<field>' ...`).
- [ ] Update inline query validation in `src/routes/account.js` for `GET /account/:id/volume` to throw `err.isValidation=true` with consistent message/field metadata.

## New endpoint: GET /account/:id/transaction-stats
- [ ] Implement the endpoint in `src/routes/account.js`.
- [ ] Add minimal query handling (if any).
- [ ] Add tests (or extend existing test coverage) to validate response shape and error handling.

## Issue #585: New Endpoint GET /account/:id/payment-summary
- [x] Add GET /:id/payment-summary route handler to `src/routes/account.js`
- [x] Add "payment-summary" to reserved words list to prevent routing conflicts
- [x] Returns { success: true, data: { totalSent, totalReceived, volumeSent, volumeReceived, topCounterparty, topAsset } }
- [x] All volume values are seven-decimal strings
- [x] Returns zeroed values for accounts with no payment history rather than a 404

## Issue #579: Add ?assets= filter to GET /account/:id/balances
- [x] Add optional ?assets= query param parsing to /balances route
- [x] "XLM" returns only native balance, "CODE:ISSUER" filters asset balances
- [x] Invalid identifiers are ignored
- [x] Returns empty array when no assets match

## Repo integrity
- [ ] Resolve merge conflict markers in `src/index.js` (currently present as `<<<<<<< HEAD` / `=======` / `>>>>>>>`).
- [ ] Ensure `npm test` passes.

## Issue #397: New Endpoint GET /transaction/:hash/effects
- [ ] Inspect existing transaction routes and response/normalization utilities
- [ ] Implement GET /transaction/:hash/effects route
  - [ ] Validate :hash is 64-char hex before Horizon call
  - [ ] Fetch all effects for transaction hash via Horizon
  - [ ] Normalize each effect with: effectId, type, account, createdAt, plus type-specific fields (best-effort)
  - [ ] Return { success: true, data: { effects: [...], total } }
  - [ ] Return 404 with clear message when transaction hash does not exist
- [x] Inspect existing transaction routes and response/normalization utilities
- [x] Implement GET /transaction/:hash/effects route
  - [x] Validate :hash is 64-char hex before Horizon call
  - [x] Fetch all effects for transaction hash via Horizon
  - [x] Normalize each effect with: effectId, type, account, createdAt, plus type-specific fields (best-effort)
  - [x] Return { success: true, data: { effects: [...], total } }
  - [x] Return 404 with clear message when transaction hash does not exist
- [x] Add/Update tests for the new endpoint (shape + validation + 404 behavior)
- [x] Ensure routing is registered in src/index.js (and docs list if applicable)
- [x] Run targeted unit tests for the endpoint only (no build)
# StellarKit API 🚀

<p align="center">
  <a href="README.md">English 🇺🇸</a> | <b>Français 🇫🇷</b> | <a href="README.es.md">Español 🇪🇸</a>
</p>

> Une API REST utilitaire pour les développeurs sur la blockchain Stellar — construite avec Express.js et le SDK Stellar officiel.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js](https://img.shields.io/badge/node-%3E%3D18.0.0-brightgreen)](https://nodejs.org/)
[![Stellar](https://img.shields.io/badge/Stellar-SDK-blue)](https://stellar.org)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

StellarKit API encapsule l'[API Horizon de Stellar](https://developers.stellar.org/api/horizon) dans des points de terminaison (endpoints) REST clairs et adaptés aux développeurs. Elle aide les développeurs créant sur Stellar à accéder rapidement aux estimations de frais, aux données de compte, à l'historique des transactions, à l'état du réseau et aux métadonnées des actifs — sans avoir à analyser les réponses brutes d'Horizon.

---

## ✨ Fonctionnalités

- 📊 **État du réseau** — Informations sur le dernier registre (ledger), frais de base, version du protocole
- 💸 **Estimation des frais** — Niveaux de frais Économique / Standard / Prioritaire pour n'importe quel nombre d'opérations
- 👤 **Informations de compte** — Soldes (XLM + tous les actifs), signataires, seuils, solde disponible (spendable balance)
- 📜 **Historique des transactions** — Transactions et opérations paginées par compte
- 🪙 **Métadonnées des actifs** — Statistiques pour n'importe quel actif Stellar, ainsi qu'une recherche multi-émetteur
- 🛡️ **Prêt pour la production** — Limitation du taux (rate limiting), en-têtes de sécurité Helmet, gestion centralisée des erreurs
- ✅ **Testé** — Suite de tests Jest avec couverture (coverage)

---

## 🚀 Démarrage rapide

### Prérequis

- Node.js >= 18
- npm >= 9

### Installation

```bash
git clone https://github.com/stellarkit-lab-devtools/stellarkit-api.git
cd stellarkit-api
npm install
cp .env.example .env
```

### Configuration

Modifiez le fichier `.env` :

```env
STELLAR_NETWORK=testnet     # ou "mainnet"
PORT=3000
```

### Exécution

```bash
# Développement (rechargement automatique)
npm run dev

# Production
npm start
```

L'API sera disponible sur `http://localhost:3000`.

---

## 📡 Points de terminaison de l'API (Endpoints)

### `GET /`
Retourne la liste complète des points de terminaison disponibles.

---

### `GET /health`
Vérification de l'état de santé du service.

**Réponse :**
```json
{
  "success": true,
  "data": {
    "status": "ok",
    "service": "StellarKit API",
    "version": "1.0.0",
    "network": "testnet"
  }
}
```

---

### `GET /network-status`
Retourne les informations du dernier registre (ledger), les frais et la version du protocole.

**Réponse :**
```json
{
  "success": true,
  "data": {
    "network": "testnet",
    "latestLedger": {
      "sequence": 123456,
      "closedAt": "2024-07-01T12:00:00Z",
      "transactionCount": 42,
      "operationCount": 89
    },
    "fees": {
      "baseFeeInStroops": 100,
      "baseFeeInXLM": "0.0000100"
    },
    "protocol": { "version": 21 }
  }
}
```

---

### `GET /fee-estimate`
Retourne les niveaux de frais Économique / Standard / Prioritaire basés sur les statistiques en direct du réseau.

**Paramètres de requête (Query params) :**
| Paramètre | Type | Par défaut | Description |
|-----------|------|------------|-------------|
| `operations` | number | `1` | Nombre d'opérations dans votre transaction |

**Exemple :**
```
GET /fee-estimate?operations=3
```

**Réponse :**
```json
{
  "success": true,
  "data": {
    "operationCount": 3,
    "perOperation": {
      "economy":  { "stroops": 100, "xlm": "0.0000100" },
      "standard": { "stroops": 200, "xlm": "0.0000200" },
      "priority": { "stroops": 500, "xlm": "0.0000500" }
    },
    "totalFee": {
      "economy":  { "stroops": 300, "xlm": "0.0000300" },
      "standard": { "stroops": 600, "xlm": "0.0000600" },
      "priority": { "stroops": 1500, "xlm": "0.0001500" }
    }
  }
}
```

---

### `GET /account/:id`
Retourne les détails complets d'un compte pour une clé publique Stellar donnée.

**Exemple :**
```
GET /account/GAAZI4TCR3TY5OJHCTJC2A4QSY6CJWJH5IAJTGKIN2ER7LBNVKOCCWN
```

**Réponse :**
```json
{
  "success": true,
  "data": {
    "accountId": "GAAZI4...",
    "sequence": "12345678",
    "xlm": {
      "balance": "100.0000000",
      "minimumBalance": "1.0000000",
      "spendableBalance": "99.0000000"
    },
    "assets": [...],
    "signers": [...],
    "flags": {...}
  }
}
```

---

### `GET /transactions/:id`
Retourne l'historique paginé des transactions d'un compte.

**Paramètres de requête (Query params) :**
| Paramètre | Type | Par défaut | Description |
|-----------|------|------------|-------------|
| `limit` | number | `10` | Nombre de résultats (max 200) |
| `order` | string | `desc` | `asc` ou `desc` |
| `cursor` | string | — | Curseur de pagination de la réponse précédente |

---

### `GET /transactions/:id/operations`
Retourne l'historique paginé des opérations d'un compte. Même paramètres de requête que ci-dessus.

---

### `GET /asset/:code/:issuer`
Retourne les métadonnées et statistiques pour un actif Stellar spécifique.

**Exemple :**
```
GET /asset/USDC/GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN
```

---

### `GET /asset/search?code=:code`
Recherche tous les actifs correspondant à un code donné à travers tous les émetteurs.

**Exemple :**
```
GET /asset/search?code=USDC
```

---

## 🧪 Exécution des tests

```bash
npm test
```

Les tests utilisent [Jest](https://jestjs.io/) + [Supertest](https://github.com/ladjs/supertest). Le rapport de couverture est généré dans `coverage/`.

---

## 🤝 Contribution

Les contributions sont les bienvenues ! Ce projet participe au programme **[Stellar Wave Program on Drips](https://www.drips.network/wave/stellar)** — vous pouvez gagner des récompenses en résolvant des issues ouvertes.

**Pour contribuer :**

1. Forkez le dépôt
2. Créez une branche de fonctionnalité : `git checkout -b feat/votre-fonctionnalite`
3. Commitez vos modifications : `git commit -m "feat: ajouter votre fonctionnalité"`
4. Poussez et ouvrez une Pull Request (PR)

Veuillez lire [CONTRIBUTING.md](CONTRIBUTING.md) avant de soumettre.

---

## 📁 Structure du Projet

```
stellarkit-api/
├── src/
│   ├── config/
│   │   └── stellar.js         # Configuration du SDK Stellar + Horizon
│   ├── middleware/
│   │   ├── errorHandler.js    # Formatage centralisé des erreurs
│   │   └── rateLimiter.js     # Limitation du taux (rate limiting)
│   ├── routes/
│   │   ├── account.js         # Points de terminaison /account
│   │   ├── asset.js           # Points de terminaison /asset
│   │   ├── feeEstimate.js     # Point de terminaison /fee-estimate
│   │   ├── networkStatus.js   # Point de terminaison /network-status
│   │   └── transactions.js    # Points de terminaison /transactions
│   ├── utils/
│   │   ├── response.js        # Utilitaires de réponse
│   │   └── validators.js      # Utilitaires de validation des entrées
│   └── index.js               # Point d'entrée de l'application
├── tests/
│   └── api.test.js
├── .env.example
├── package.json
└── README.md
```

---

## 🌐 Ressources Stellar

- [Portail des Développeurs Stellar](https://developers.stellar.org)
- [SDK JavaScript Stellar](https://github.com/stellar/js-stellar-sdk)
- [Référence de l'API Horizon](https://developers.stellar.org/api/horizon)
- [Discord Stellar](https://discord.gg/stellardev)
- [Programme Stellar Wave](https://www.drips.network/wave/stellar)

---

## 📄 Licence

[MIT](LICENSE)


