# Veille Immobilière Automatisée — Multiplex Montréal

Workflow n8n qui analyse automatiquement les alertes immobilières reçues par email (Centris, DuProprio, Realtor) et génère un rapport d'investissement quotidien grâce à l'IA.

## Ce que ça fait

1. **Trigger** — Se déclenche chaque jour à 17h sur un label Gmail spécifique
2. **Extraction** — Parse les emails d'alerte des plateformes immobilières
3. **Contexte historique** — Récupère les 30 dernières propriétés analysées depuis Google Sheets
4. **Analyse IA** — Envoie les données à Claude (Anthropic) qui calcule les métriques financières et attribue un score /10
5. **Stockage** — Sauvegarde chaque propriété analysée dans Google Sheets
6. **Rapport** — Envoie un email HTML récapitulatif uniquement si une propriété score ≥ 7/10

## Critères d'investissement ciblés

| Paramètre | Valeur |
|---|---|
| Budget | 700 000 $ – 1 000 000 $ |
| Types | Triplex, Quadruplex, Quintuplex |
| Zone | Île de Montréal |
| Mise de fonds | 10 % (propriétaire-occupant) |
| Taux stressé | 5,75 %, amortissement 25 ans |

## Métriques calculées par l'IA

- **GRM** (Gross Rent Multiplier) — Prix / revenus bruts annuels. Seuil : < 14 excellent, 14–17 acceptable
- **Cap rate** — NOI / prix (hypothèse : 35 % de dépenses)
- **DSCR** — Couverture du service de la dette. Seuil : > 1,0
- **Coût net d'habiter** — Paiement hypothécaire + taxes − revenus locatifs des autres unités

## Stack technique

- **[n8n](https://n8n.io)** — Orchestration du workflow (self-hosted via Docker)
- **Gmail** — Source des alertes + envoi du rapport
- **Claude API** (Anthropic) — Analyse et scoring des propriétés
- **Google Sheets** — Historique et tracker des propriétés analysées

## Prérequis

- Docker & Docker Compose
- Compte n8n (self-hosted ou cloud)
- Clé API Anthropic
- Compte Gmail avec OAuth2 configuré dans n8n
- Compte Google Sheets avec OAuth2 configuré dans n8n

## Installation

### 1. Cloner le repo

```bash
git clone https://github.com/Danforthhh/n8n-immobilier.git
cd n8n-immobilier
```

### 2. Configurer les variables d'environnement

```bash
cp .env.txt .env
```

Remplir `.env` :

```env
N8N_ENCRYPTION_KEY=une_clé_aléatoire_longue
```

### 3. Lancer n8n

```bash
docker-compose up -d
```

n8n sera accessible sur [http://localhost:5678](http://localhost:5678).

### 4. Importer le workflow

1. Dans n8n → **Workflows** → **Import from file**
2. Sélectionner `workflows/veille-immobiliere.json.json`
3. Reconfigurer les credentials (Gmail, Google Sheets, Anthropic)

### 5. Configurer les valeurs personnelles

Se référer à `config.example.json` pour les paramètres à adapter :

| Paramètre | Où le modifier dans n8n |
|---|---|
| Emails destinataires | Nœud **Send a message** → champ `sendTo` |
| ID Google Sheets | Nœuds **Get row(s)** et **Append or update** → sélectionner ton fichier |
| Label Gmail | Nœud **Gmail Trigger** → champ `labelIds` |

### 6. Configurer Gmail

Créer un label Gmail (ex. `Alertes Immobilières`) et y faire rediriger automatiquement les emails de Centris, DuProprio et Realtor via les filtres Gmail.

## Structure du projet

```
n8n-immobilier/
├── docker-compose.yml          # Configuration Docker pour n8n
├── workflows/
│   └── veille-immobiliere.json.json  # Workflow n8n à importer
├── config.example.json         # Template des valeurs à personnaliser
├── .env.txt                    # Template des variables d'environnement
└── .gitignore
```

## Personnalisation

Le prompt envoyé à Claude est entièrement configurable dans le nœud **Code3** du workflow. Tu peux y modifier :

- Les quartiers prioritaires
- Les seuils de GRM / DSCR
- Le budget
- Les types de propriétés recherchés
- Le format du rapport email

## Licence

MIT
