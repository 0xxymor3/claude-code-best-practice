# Guide Complet de Claude Code 🇫🇷

*Basé sur le dépôt [claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) — de l'ingénierie vibe à l'ingénierie agentique.*

---

## Table des matières

1. [Qu'est-ce que Claude Code ?](#1-quest-ce-que-claude-code-)
2. [Installation et premiers pas](#2-installation-et-premiers-pas)
3. [La Mémoire — CLAUDE.md et Rules](#3-la-mémoire--claudemd-et-rules)
4. [Configuration — settings.json](#4-configuration--settingsjson)
5. [Les Agents (Subagents)](#5-les-agents-subagents)
6. [Les Compétences (Skills)](#6-les-compétences-skills)
7. [Les Commandes (Slash Commands)](#7-les-commandes-slash-commands)
8. [Quand utiliser Agent vs Command vs Skill ?](#8-quand-utiliser-agent-vs-command-vs-skill-)
9. [Architecture d'orchestration — Command → Agent → Skill](#9-architecture-dorchestration--command--agent--skill)
10. [Les Hooks](#10-les-hooks)
11. [Le Protocole MCP (Model Context Protocol)](#11-le-protocole-mcp-model-context-protocol)
12. [Gestion du contexte](#12-gestion-du-contexte)
13. [Fonctionnalités avancées](#13-fonctionnalités-avancées)
14. [Conseils pratiques](#14-conseils-pratiques)
15. [Dépannage](#15-dépannage)
16. [Ressources](#16-ressources)

---

## 1. Qu'est-ce que Claude Code ?

Claude Code est l'interface en ligne de commande (CLI) officielle d'Anthropic pour interagir avec Claude directement depuis votre terminal. Ce n'est pas simplement un outil de complétion de code : c'est un agent de développement autonome capable de lire, écrire, exécuter et déboguer du code, de gérer des dépôts Git, d'appeler des APIs externes et d'orchestrer des flux de travail complexes.

### De l'ingénierie vibe à l'ingénierie agentique

La progression décrite dans ce dépôt reflète une évolution dans la façon dont les développeurs utilisent l'IA :

- **Ingénierie vibe** : on demande à Claude de générer du code "dans l'esprit" de ce qu'on veut, on corrige manuellement, on itère. C'est rapide mais pas systématique.
- **Ingénierie agentique** : on définit des agents, des skills, des hooks et des commandes pour que Claude agisse de façon autonome et répétable sur des tâches complexes — avec isolation du contexte, mémoire persistante, et vérification automatique.

La ligne de démarcation n'est pas dans l'outil mais dans la façon de l'utiliser : donner à Claude les moyens de vérifier son propre travail, d'apprendre de ses erreurs et de fonctionner pendant des heures sans intervention humaine.

### Les 3 piliers d'extension

Claude Code s'étend selon trois mécanismes complémentaires :

| Pilier | Fichier | Rôle |
|--------|---------|------|
| **Agent** | `.claude/agents/<name>.md` | Contexte isolé, multi-étapes, autonome |
| **Command** | `.claude/commands/<name>.md` | Point d'entrée initié par l'utilisateur |
| **Skill** | `.claude/skills/<name>/SKILL.md` | Procédure réutilisable, invocable inline |

### Vue d'ensemble de l'architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      CLAUDE CODE                            │
│                                                             │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────────┐  │
│  │   Commands  │   │   Agents    │   │     Skills      │  │
│  │  /cmd-name  │──▶│ contexte    │──▶│  procédures     │  │
│  │  (user)     │   │ isolé       │   │  réutilisables  │  │
│  └─────────────┘   └─────────────┘   └─────────────────┘  │
│         │                 │                   │             │
│         ▼                 ▼                   ▼             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   CLAUDE.md + Rules                 │   │
│  │            (mémoire persistante du projet)          │   │
│  └─────────────────────────────────────────────────────┘   │
│         │                                                   │
│         ▼                                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Hooks + MCP + Settings                 │   │
│  │        (automatisation + outils + permissions)      │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

> **💡 Conseil :** Claude Code fonctionne très bien dès l'installation. N'essayez pas de tout configurer d'emblée — commencez avec un bon CLAUDE.md, ajoutez des commandes pour vos workflows répétitifs, puis évoluez vers des agents quand vous avez besoin d'autonomie.

---

## 2. Installation et premiers pas

### Prérequis

- **Node.js 18+** — Claude Code est distribué via npm et requiert Node.js version 18 ou supérieure
- Un terminal (macOS, Linux, ou Windows avec WSL)
- Un compte Anthropic (Claude Pro, Max, ou API key)

Vérifiez votre version de Node.js :

```bash
node --version  # doit afficher v18.x ou plus
```

### Installation

```bash
npm install -g @anthropic-ai/claude-code
```

Vérifiez l'installation :

```bash
claude --version
```

### Authentification

Lancez Claude Code pour la première fois :

```bash
claude
```

Trois méthodes d'authentification sont disponibles :

**Méthode 1 — Abonnement Claude.ai (Pro / Max)**
- Sélectionnez "Claude.ai account"
- Le navigateur s'ouvre pour l'autorisation OAuth
- Revenez au terminal — vous êtes connecté

**Méthode 2a — API Key (invitation d'équipe)**
- Votre admin vous invite depuis le dashboard Anthropic
- Vous recevez un email d'invitation — acceptez-le
- Lancez `claude` → "Anthropic API Key" → la clé est auto-générée

**Méthode 2b — API Key (vous avez la clé)**
- Lancez `claude` → "Anthropic API Key"
- Collez votre clé (commence par `sk-ant-`)
- La clé est stockée de façon permanente

Vous pouvez aussi vous déconnecter et reconnecter à tout moment avec `/login` et `/logout`.

### Structure du répertoire `.claude/`

Voici l'arborescence typique d'un projet configuré :

```
mon-projet/
├── CLAUDE.md                    # Mémoire principale du projet
├── .claude/
│   ├── settings.json            # Configuration partagée de l'équipe
│   ├── settings.local.json      # Préférences personnelles (git-ignoré)
│   ├── agents/                  # Agents personnalisés
│   │   └── mon-agent.md
│   ├── commands/                # Commandes slash personnalisées
│   │   └── ma-commande.md
│   ├── skills/                  # Skills réutilisables
│   │   └── mon-skill/
│   │       └── SKILL.md
│   ├── rules/                   # Règles chargées conditionnellement
│   │   └── ma-regle.md
│   └── hooks/                   # Scripts de lifecycle
│       ├── scripts/
│       └── config/
└── .mcp.json                    # Serveurs MCP du projet
```

> **💡 Conseil :** Exécutez `/init` pour générer automatiquement un CLAUDE.md initial en analysant votre projet. Avec `CLAUDE_CODE_NEW_INIT=1`, le flux interactif vous guide aussi pour configurer les skills, hooks et fichiers de mémoire personnelle.

---

## 3. La Mémoire — CLAUDE.md et Rules

La mémoire de Claude Code repose sur des fichiers Markdown que le modèle lit au démarrage (ou de façon conditionnelle). Comprendre le mécanisme de chargement est essentiel pour organiser correctement ses instructions.

### Ce qu'est un CLAUDE.md

Un fichier `CLAUDE.md` est un fichier Markdown ordinaire qui contient des instructions permanentes pour Claude. Il peut contenir :
- Les conventions de code de l'équipe
- Les formats de messages de commit
- Les commandes de build et de test
- Les pièges à éviter (les fameuses "gotchas")
- Les décisions d'architecture importantes

La règle d'or : **ce que vous écrivez dans CLAUDE.md, Claude s'en souvient toujours** — sans que vous ayez à le répéter à chaque session.

### Mécanisme de chargement — ancêtres et descendants

Claude Code utilise deux mécanismes distincts pour charger les fichiers CLAUDE.md :

#### Chargement ascendant (ancêtres — au démarrage)

Au lancement, Claude remonte l'arbre des répertoires depuis le répertoire courant jusqu'à la racine du système de fichiers, et charge tous les fichiers CLAUDE.md trouvés. Ces fichiers sont **chargés immédiatement au démarrage**.

#### Chargement descendant (descendants — paresseux)

Les fichiers CLAUDE.md dans les sous-répertoires ne sont **pas** chargés au lancement. Ils sont inclus uniquement lorsque Claude lit des fichiers dans ces sous-répertoires pendant la session. C'est le **chargement paresseux** (lazy loading).

#### Les frères et sœurs ne se chargent jamais

Si vous travaillez dans `frontend/`, vous n'obtiendrez jamais `backend/CLAUDE.md` dans votre contexte.

### Tableau des scénarios en monorepo

Considérons cette structure :

```
/monorepo/
├── CLAUDE.md          # Instructions partagées
├── frontend/
│   └── CLAUDE.md      # Instructions frontend
├── backend/
│   └── CLAUDE.md      # Instructions backend
└── api/
    └── CLAUDE.md      # Instructions API
```

**Scénario 1 : lancement depuis `/monorepo/`**

| Fichier | Chargé au démarrage ? | Raison |
|---------|----------------------|--------|
| `/monorepo/CLAUDE.md` | Oui | Répertoire courant |
| `/monorepo/frontend/CLAUDE.md` | Non | Chargé uniquement si vous touchez des fichiers dans `frontend/` |
| `/monorepo/backend/CLAUDE.md` | Non | Chargé uniquement si vous touchez des fichiers dans `backend/` |
| `/monorepo/api/CLAUDE.md` | Non | Chargé uniquement si vous touchez des fichiers dans `api/` |

**Scénario 2 : lancement depuis `/monorepo/frontend/`**

| Fichier | Chargé au démarrage ? | Raison |
|---------|----------------------|--------|
| `/monorepo/CLAUDE.md` | Oui | Ancêtre — toujours chargé |
| `/monorepo/frontend/CLAUDE.md` | Oui | Répertoire courant |
| `/monorepo/backend/CLAUDE.md` | Non | Frère — jamais chargé |
| `/monorepo/api/CLAUDE.md` | Non | Frère — jamais chargé |

### `.claude/rules/` — chargement conditionnel avec `paths:`

Les fichiers dans `.claude/rules/` offrent un contrôle plus fin. Avec un frontmatter `paths:`, un fichier de règles ne se charge que lorsque Claude travaille sur des fichiers correspondants :

```yaml
---
paths:
  - "src/api/**"
  - "*.graphql"
---

# Règles API GraphQL

Toujours valider les mutations avec zod avant de les envoyer.
```

Sans frontmatter `paths:`, le fichier se charge dans toutes les sessions comme un CLAUDE.md supplémentaire.

### `~/.claude/CLAUDE.md` — mémoire globale

Ce fichier s'applique à toutes vos sessions Claude Code, quel que soit le projet. Il est idéal pour vos préférences personnelles qui n'appartiennent pas au dépôt de l'équipe : votre style de communication préféré, vos conventions personnelles, vos raccourcis habituels.

### Bonnes pratiques pour écrire un CLAUDE.md

1. **Gardez chaque fichier sous 200 lignes** — au-delà, l'adhérence de Claude diminue
2. **Mettez les conventions partagées à la racine** — standards de commit, architecture globale
3. **Mettez les instructions spécifiques au composant dans le sous-répertoire** — frameworks, patterns de test propres à ce module
4. **Utilisez `CLAUDE.local.md`** (git-ignoré) pour vos préférences personnelles qui ne doivent pas être partagées avec l'équipe
5. **Mettez à jour fréquemment** — chaque fois que Claude fait quelque chose d'incorrect, ajoutez une règle pour éviter que cela se reproduise
6. **Formulez les interdictions clairement** : "Ne jamais utiliser `var`, toujours `const` ou `let`"

> **💡 Conseil :** Boris Cherny recommande que toute l'équipe contribue au CLAUDE.md plusieurs fois par semaine. Tagger `@claude` dans les PR pour demander des mises à jour du CLAUDE.md est une excellente habitude — c'est de l'ingénierie composée.

> **⚠️ Attention :** Les fichiers `.claude/rules/*.md` sans frontmatter `paths:` sont chargés dans toutes les sessions, comme un CLAUDE.md global. Avec le frontmatter `paths:`, ils sont chargés de façon paresseuse uniquement quand vous touchez des fichiers correspondants — ce qui est beaucoup plus économique en contexte.

---

## 4. Configuration — settings.json

Le fichier `.claude/settings.json` est la colonne vertébrale de la configuration de Claude Code. Il contrôle les permissions, le modèle, le style de sortie, les hooks et bien plus encore.

### La hiérarchie à 5 niveaux

Les paramètres s'appliquent dans cet ordre de priorité (du plus prioritaire au moins prioritaire) :

| Priorité | Emplacement | Portée | Versionnement | Usage |
|----------|-------------|--------|---------------|-------|
| 0 | `managed-settings.json` / MDM / Registre | Organisation | Géré par l'org | Politiques imposées, non modifiables |
| 1 | Arguments de ligne de commande | Session | N/A | Remplacements ponctuels |
| 2 | `.claude/settings.local.json` | Projet | Non (git-ignoré) | Préférences personnelles sur le projet |
| 3 | `.claude/settings.json` | Projet | Oui (committé) | Configuration partagée de l'équipe |
| 4 | `~/.claude/settings.local.json` | Global | N/A | Remplacements personnels globaux |
| 5 | `~/.claude/settings.json` | Global | N/A | Paramètres personnels globaux |

> **⚠️ Attention :** Les règles `deny` ont la priorité de sécurité la plus haute — elles ne peuvent pas être remplacées par des règles `allow` ou `ask` de priorité inférieure.

### Permissions — allow / ask / deny

Le système de permissions contrôle quels outils Claude peut utiliser sans confirmation :

```json
{
  "permissions": {
    "allow": [
      "Edit(*)",
      "Write(*)",
      "Bash(*)",
      "WebFetch(domain:api.open-meteo.com)",
      "mcp__playwright__*"
    ],
    "ask": [
      "Bash(rm *)",
      "Bash(docker *)",
      "Bash(npm *)"
    ],
    "deny": [
      "Bash(rm -rf /)"
    ]
  }
}
```

La syntaxe wildcard est complète : `Bash(bun run *)`, `Edit(/docs/**)`, `mcp__*`.

### Configuration du modèle

```json
{
  "model": "claude-sonnet-4-6",
  "env": {
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "80"
  }
}
```

### Style de sortie

```json
{
  "outputStyle": "Explanatory"
}
```

Les styles disponibles incluent : `Explanatory` (recommandé pour découvrir un nouveau codebase — Claude explique les frameworks et patterns en travaillant), `Learning` (Claude vous coache pour faire les changements vous-même), ou un style personnalisé.

### Ligne de statut

```json
{
  "statusLine": {
    "type": "command",
    "command": "echo \"$(git branch --show-current) | $(date +'%H:%M')\"",
    "padding": 0
  }
}
```

Exécutez `/statusline` pour laisser Claude générer automatiquement une ligne de statut à partir de votre `.bashrc`/`.zshrc`.

### Compaction automatique

```json
{
  "env": {
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "80"
  }
}
```

La compaction automatique se déclenche quand le contexte atteint le pourcentage configuré (80% par défaut).

### Attribution dans les commits

```json
{
  "attribution": {
    "commit": "Co-Authored-By: Claude <noreply@anthropic.com>",
    "pr": "Generated with [Claude Code](https://claude.ai/code)"
  }
}
```

### Verbes et conseils du spinner

Personnalisez les messages affichés pendant que Claude réfléchit :

```json
{
  "spinnerVerbs": {
    "mode": "replace",
    "verbs": ["Analyse...", "Réflexion en cours...", "Architecture en construction..."]
  },
  "spinnerTipsOverride": {
    "tips": ["Conseil #1 de votre équipe", "Conseil #2"],
    "excludeDefault": true
  }
}
```

> **💡 Conseil :** Committez votre `.claude/settings.json` dans git pour que toute l'équipe bénéficie des mêmes configurations. Utilisez `.claude/settings.local.json` (git-ignoré) pour vos personnalisations individuelles.

---

## 5. Les Agents (Subagents)

Un agent (subagent) est une instance autonome de Claude avec son propre contexte isolé. Contrairement à une simple invocation inline, un agent peut effectuer des dizaines d'étapes en autonomie, utiliser un ensemble d'outils restreint, et maintenir une mémoire persistante entre les sessions.

### Concept fondamental

Quand Claude invoque un agent, il crée une nouvelle fenêtre de contexte indépendante — comme ouvrir un nouveau terminal. L'agent reçoit ses instructions, travaille en autonomie, puis retourne son résultat. Le contexte principal n'est pas pollué par les travaux intermédiaires de l'agent.

### Fichier de définition

Les agents se définissent dans `.claude/agents/<name>.md` avec un frontmatter YAML suivi du corps de l'instruction.

### Les 16 champs frontmatter des agents

| Champ | Type | Requis | Description |
|-------|------|--------|-------------|
| `name` | string | Oui | Identifiant unique en minuscules avec tirets |
| `description` | string | Oui | Quand invoquer cet agent. Utilisez `"PROACTIVELY"` pour l'auto-invocation par Claude |
| `tools` | string/list | Non | Liste blanche d'outils (ex: `Read, Write, Edit, Bash`). Hérite de tous les outils si omis. Supporte la syntaxe `Agent(agent_type)` |
| `disallowedTools` | string/list | Non | Outils à interdire, retirés de la liste héritée ou spécifiée |
| `model` | string | Non | Modèle à utiliser : `sonnet`, `opus`, `haiku`, un ID complet (ex: `claude-opus-4-6`), ou `inherit` (défaut) |
| `permissionMode` | string | Non | Mode de permission : `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, ou `plan` |
| `maxTurns` | integer | Non | Nombre maximum de tours agentiques avant l'arrêt |
| `skills` | list | Non | Noms de skills à précharger dans le contexte de l'agent au démarrage |
| `mcpServers` | list | Non | Serveurs MCP pour cet agent — noms de serveurs ou objets de configuration inline |
| `hooks` | object | Non | Hooks de lifecycle scoped à cet agent |
| `memory` | string | Non | Portée de la mémoire persistante : `user`, `project`, ou `local` |
| `background` | boolean | Non | `true` pour toujours exécuter en tâche de fond (défaut: `false`) |
| `effort` | string | Non | Niveau d'effort : `low`, `medium`, `high`, `xhigh`, `max` (Opus 4.6 uniquement) |
| `isolation` | string | Non | `"worktree"` pour exécuter dans un git worktree temporaire |
| `initialPrompt` | string | Non | Soumis automatiquement comme premier tour utilisateur quand cet agent est l'agent principal de session |
| `color` | string | Non | Couleur d'affichage : `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, ou `cyan` |

### Les 5 agents officiels

| # | Agent | Modèle | Outils | Description |
|---|-------|--------|--------|-------------|
| 1 | `general-purpose` | inherit | Tous | Tâches complexes multi-étapes — type d'agent par défaut pour la recherche, l'exploration de code et le travail autonome |
| 2 | `Explore` | haiku | Lecture seule (sans Write, Edit) | Recherche et exploration rapide du codebase — optimisé pour trouver des fichiers et répondre aux questions sur le code |
| 3 | `Plan` | inherit | Lecture seule | Recherche pré-planification en mode plan — explore le codebase et conçoit des approches d'implémentation avant d'écrire du code |
| 4 | `statusline-setup` | sonnet | Read, Edit | Configure le paramètre de ligne de statut de l'utilisateur |
| 5 | `claude-code-guide` | haiku | Glob, Grep, Read, WebFetch, WebSearch | Répond aux questions sur les fonctionnalités de Claude Code, l'Agent SDK et l'API Claude |

### Exemple d'agent complet

```yaml
---
name: code-reviewer
description: Use this agent PROACTIVELY to review code changes for quality and best practices
tools: Read, Bash
model: sonnet
maxTurns: 10
permissionMode: acceptEdits
memory: user
color: blue
---

# Code Reviewer

Tu es un expert en revue de code. Avant de commencer, consulte ta mémoire
pour les patterns que tu as déjà observés dans ce projet.

## Contrat d'exécution (non négociable)

Tu DOIS :
1. Lire les fichiers modifiés avant de commenter
2. Mettre à jour ta mémoire avec les patterns nouveaux découverts

Tu es INTERDIT de :
- Proposer des refactorisations non demandées
- Changer le comportement fonctionnel du code
```

### Mémoire persistante des agents

Le champ `memory` donne à chaque agent son propre store de connaissance en Markdown qui persiste entre les sessions :

| Portée | Emplacement | Versionné | Partagé | Idéal pour |
|--------|-------------|-----------|---------|------------|
| `user` | `~/.claude/agent-memory/<agent-name>/` | Non | Non | Connaissance cross-projet (recommandé par défaut) |
| `project` | `.claude/agent-memory/<agent-name>/` | Oui | Oui | Connaissance projet partagée avec l'équipe |
| `local` | `.claude/agent-memory-local/<agent-name>/` | Non (git-ignoré) | Non | Connaissance personnelle et projet-spécifique |

Au démarrage, les 200 premières lignes de `MEMORY.md` sont injectées dans le prompt système de l'agent. Si `MEMORY.md` dépasse 200 lignes, l'agent déplace les détails dans des fichiers thématiques.

### Pattern "Execution Contracts" (contrats d'exécution)

Un pattern puissant recommandé dans ce dépôt est de définir explicitement ce que l'agent DOIT faire et ce qu'il est INTERDIT de faire :

```
## Contrat d'exécution (non négociable)

Tu DOIS compléter cette tâche en déléguant à l'agent X. Tu es INTERDIT de :
- Fetcher des données toi-même via Bash, WebFetch, ou tout autre outil
- Sauter l'étape 1 (la préférence utilisateur est une entrée requise)
- Appeler l'étape 3 avant que l'agent retourne un résultat
```

### Pattern "Learnings" (agents auto-évolutifs)

En combinant `memory:` avec des instructions explicites, les agents apprennent au fil du temps :

```yaml
memory: project
---
Avant de commencer : lis ta mémoire pour les patterns connus.
Après avoir terminé : mets à jour ta mémoire avec ce que tu as appris.
```

> **💡 Conseil :** Les subagents ne peuvent **pas** invoquer d'autres subagents via des commandes bash. Utilisez l'outil `Agent` (anciennement `Task`) : `Agent(subagent_type="mon-agent", prompt="...")`. Soyez explicite sur l'usage des outils dans les définitions de subagents — évitez les termes vagues comme "lancer" qui pourraient être interprétés comme des commandes bash.

> **⚠️ Attention :** Le champ `tools` définit une liste blanche. Si vous l'omettez, l'agent hérite de tous les outils. Si vous avez besoin de restreindre un agent réseau-only, spécifiez explicitement `tools: WebFetch, Read`.

---

## 6. Les Compétences (Skills)

Un skill est une procédure réutilisable qui peut être invoquée de trois façons : par l'utilisateur via `/skill-name`, automatiquement par Claude quand le contexte le justifie, ou préchargée dans un agent comme connaissance de domaine.

### Concept fondamental

À la différence des agents, un skill s'exécute dans le contexte de la conversation principale (à moins d'utiliser `context: fork`). Il ne crée pas une nouvelle fenêtre de contexte — il ajoute des instructions inline au flux courant. C'est plus léger et plus rapide qu'un agent pour des tâches procédurales bien définies.

La distinction clé : les skills sont des **dossiers**, pas seulement des fichiers Markdown. Un skill peut contenir des scripts, des assets, des données de référence, des exemples — tout ce que l'agent peut découvrir et exploiter.

### Fichier de définition

Les skills se définissent dans `.claude/skills/<name>/SKILL.md`. Le dossier peut contenir des fichiers supplémentaires référencés dans le SKILL.md.

### Les 16 champs frontmatter des skills

| Champ | Type | Requis | Description |
|-------|------|--------|-------------|
| `name` | string | Non | Nom d'affichage et identifiant `/slash-command`. Par défaut : nom du répertoire |
| `description` | string | Recommandé | Ce que fait le skill. Affiché dans l'autocomplétion et utilisé par Claude pour l'auto-découverte |
| `when_to_use` | string | Non | Contexte supplémentaire sur quand invoquer le skill — phrases déclencheurs. Concaténé à `description`, compte dans la limite de 1 536 caractères |
| `argument-hint` | string | Non | Indice affiché pendant l'autocomplétion (ex: `[issue-number]`, `[filename]`) |
| `arguments` | string/list | Non | Arguments positionnels nommés pour la substitution `$name` dans le contenu du skill |
| `disable-model-invocation` | boolean | Non | `true` pour empêcher Claude d'invoquer automatiquement ce skill |
| `user-invocable` | boolean | Non | `false` pour masquer du menu `/` — le skill devient connaissance de fond uniquement |
| `allowed-tools` | string | Non | Outils autorisés sans confirmations quand ce skill est actif |
| `disallowed-tools` | string/list | Non | Outils retirés du pool disponible pendant l'exécution du skill |
| `model` | string | Non | Modèle à utiliser quand ce skill est exécuté (`haiku`, `sonnet`, `opus`) |
| `effort` | string | Non | Niveau d'effort override lors de l'invocation (`low`, `medium`, `high`, `xhigh`, `max`) |
| `context` | string | Non | `fork` pour exécuter dans un contexte de subagent isolé |
| `agent` | string | Non | Type de subagent quand `context: fork` est défini (défaut: `general-purpose`) |
| `hooks` | object | Non | Hooks de lifecycle scoped à ce skill |
| `paths` | string/list | Non | Patterns glob qui limitent quand le skill s'active automatiquement |
| `shell` | string | Non | Shell pour les blocs `` !`command` `` — `bash` (défaut) ou `powershell` |

### Les 10 skills officiels (bundled)

| # | Skill | Description |
|---|-------|-------------|
| 1 | `code-review` | Revue du diff courant pour bugs de correction à un niveau d'effort choisi. `--comment` poste les résultats en commentaires inline de PR |
| 2 | `batch` | Exécution de commandes sur plusieurs fichiers en masse — idéal pour les migrations de code à grande échelle |
| 3 | `debug` | Débogage de commandes ou problèmes de code qui échouent |
| 4 | `loop` | Exécution d'un prompt ou d'une slash command sur un intervalle récurrent (jusqu'à 3 jours) |
| 5 | `claude-api` | Construction d'apps avec l'API Claude ou le SDK Anthropic — se déclenche sur les imports `anthropic` / `@anthropic-ai/sdk` |
| 6 | `fewer-permission-prompts` | Scanne les transcripts pour les appels Bash/MCP en lecture seule et ajoute une liste d'autorisation dans `.claude/settings.json` |
| 7 | `run` | Lance et pilote l'application du projet pour voir un changement fonctionner dans la vraie app |
| 8 | `verify` | Build et lance l'app pour confirmer qu'un changement de code fait ce qu'il devrait |
| 9 | `run-skill-generator` | Apprend à `/run` et `/verify` comment builder et lancer le projet |
| 10 | `simplify` | Revue du code modifié pour les opportunités de nettoyage (réutilisation, simplification, efficacité) — 4 agents de revue en parallèle |

### Deux patterns d'invocation

**Pattern 1 — Agent skills (préchargés)**

Le skill est injecté dans le contexte de l'agent au démarrage comme connaissance de domaine. L'agent n'invoque pas le skill via un outil — il suit ses instructions directement :

```yaml
# Dans la définition de l'agent
---
name: mon-agent
skills:
  - mon-skill   # Contenu complet injecté au démarrage
---
```

**Pattern 2 — Skills (invocation directe)**

Le skill est invoqué via l'outil Skill depuis une commande, un agent, ou Claude lui-même :

```
Skill(skill: "mon-skill")
```

### Skills background-only

Le champ `user-invocable: false` cache un skill du menu `/` tout en le rendant disponible pour le préchargement dans des agents ou l'invocation programmatique :

```yaml
---
name: weather-fetcher
description: Instructions pour récupérer la météo de Dubai via Open-Meteo
user-invocable: false
allowed-tools:
  - "WebFetch(*)"
---
```

> **💡 Conseil :** Le champ `description` n'est pas un résumé — c'est une description de **quand déclencher** ce skill. Écrivez-le pour le modèle, pas pour l'humain. Claude scanne toutes les descriptions de skills pour décider lequel invoquer sur une requête donnée.

> **💡 Conseil :** Les skills sont des dossiers. Exploitez cela : organisez les références, exemples et scripts dans des sous-fichiers que Claude peut lire progressivement. Une section "Gotchas" est souvent le contenu le plus utile d'un skill.

---

## 7. Les Commandes (Slash Commands)

Une commande est un point d'entrée que l'utilisateur déclenche explicitement. Les commandes ne sont **jamais auto-invoquées** par Claude — elles ne peuvent être appelées que par l'utilisateur via `/nom-de-commande`. Elles servent à orchestrer des workflows complexes, à interagir avec l'utilisateur, et à coordonner agents et skills.

### Concept fondamental

La commande est le chef d'orchestre. Là où un agent travaille en autonomie dans son coin, la commande est le point de contact entre l'utilisateur et le système. Elle peut demander des confirmations, passer des données entre agents et skills, et gérer le flux d'exécution global.

Le contenu d'une commande n'est pas injecté dans le contexte de la session tant que l'utilisateur ne la déclenche pas — ce qui garde le contexte lean.

### Fichier de définition

Les commandes se définissent dans `.claude/commands/<name>.md`.

### Les 16 champs frontmatter des commandes

| Champ | Type | Requis | Description |
|-------|------|--------|-------------|
| `name` | string | Non | Nom d'affichage et identifiant `/slash-command`. Par défaut : nom du fichier |
| `description` | string | Recommandé | Ce que fait la commande. Affiché dans l'autocomplétion |
| `when_to_use` | string | Non | Contexte supplémentaire sur quand utiliser cette commande |
| `argument-hint` | string | Non | Indice affiché pendant l'autocomplétion |
| `arguments` | string/list | Non | Arguments positionnels nommés pour la substitution `$name` |
| `disable-model-invocation` | boolean | Non | `true` pour empêcher l'invocation automatique (toujours false pour les commandes) |
| `user-invocable` | boolean | Non | `false` pour masquer du menu `/` |
| `paths` | string/list | Non | Patterns glob qui limitent quand ce skill s'active |
| `allowed-tools` | string | Non | Outils autorisés sans confirmations quand cette commande est active |
| `disallowed-tools` | string/list | Non | Outils retirés du pool pendant l'exécution. Se réinitialise au prochain message |
| `model` | string | Non | Modèle à utiliser quand cette commande s'exécute |
| `effort` | string | Non | Niveau d'effort override lors de l'invocation |
| `context` | string | Non | `fork` pour exécuter dans un contexte de subagent isolé |
| `agent` | string | Non | Type de subagent quand `context: fork` est défini |
| `shell` | string | Non | Shell pour les blocs `` !`command` `` — `bash` (défaut) ou `powershell` |
| `hooks` | object | Non | Hooks de lifecycle scoped à cette commande |

### Les 83 commandes officielles par catégorie

**Auth (5)**

| Commande | Description |
|----------|-------------|
| `/login` | Se connecter à votre compte Anthropic |
| `/logout` | Se déconnecter |
| `/setup-bedrock` | Configurer l'authentification Amazon Bedrock |
| `/setup-vertex` | Configurer l'authentification Google Vertex AI |
| `/upgrade` | Ouvrir la page de mise à niveau du plan |

**Config (15)**

| Commande | Description |
|----------|-------------|
| `/color [color\|default]` | Définir la couleur de la barre de prompt pour la session |
| `/config` | Ouvrir l'interface Paramètres (alias: `/settings`) |
| `/focus` | Basculer la vue focus — affiche uniquement le dernier prompt et la réponse finale |
| `/keybindings` | Ouvrir ou créer le fichier de configuration des raccourcis clavier |
| `/permissions` | Gérer les règles allow/ask/deny (alias: `/allowed-tools`) |
| `/privacy-settings` | Voir et mettre à jour vos paramètres de confidentialité |
| `/radio` | Ouvrir la radio lo-fi Claude FM dans le navigateur |
| `/sandbox` | Basculer le mode sandbox |
| `/scroll-speed` | Ajuster la vitesse de défilement |
| `/statusline` | Configurer la ligne de statut de Claude Code |
| `/stickers` | Commander des stickers Claude Code |
| `/terminal-setup` | Configurer les raccourcis clavier du terminal |
| `/theme` | Changer le thème de couleur |
| `/tui [default\|fullscreen]` | Définir le rendu TUI et relancer Claude Code |
| `/voice [hold\|tap\|off]` | Activer/désactiver la dictée vocale |

**Context (7)**

| Commande | Description |
|----------|-------------|
| `/context` | Visualiser l'utilisation du contexte courant |
| `/cost` | Alias pour `/usage` |
| `/insights` | Rapport d'analyse de vos sessions Claude Code |
| `/stats` | Alias pour `/usage` (onglet Stats) |
| `/status` | Ouvrir les Paramètres (onglet Statut) |
| `/usage` | Afficher le coût de session, limites du plan, stats d'activité |
| `/usage-credits` | Configurer les crédits d'usage |

**Debug (8)**

| Commande | Description |
|----------|-------------|
| `/doctor` | Diagnostiquer et vérifier votre installation Claude Code |
| `/feedback [report]` | Soumettre un retour ou signaler un bug |
| `/heapdump` | Écrire un snapshot heap JavaScript pour diagnostiquer la mémoire |
| `/help` | Afficher l'aide et les commandes disponibles |
| `/powerup` | Découvrir les fonctionnalités via des leçons interactives animées |
| `/release-notes` | Voir le changelog dans un sélecteur de version interactif |
| `/tasks` | Lister et gérer les tâches en arrière-plan (alias: `/bashes`) |

**Export (2)**

| Commande | Description |
|----------|-------------|
| `/copy [N]` | Copier la dernière réponse dans le presse-papiers |
| `/export [filename]` | Exporter la conversation en texte brut |

**Extensions (8)**

| Commande | Description |
|----------|-------------|
| `/agents` | Gérer les configurations d'agents |
| `/chrome` | Configurer les paramètres de Claude dans Chrome |
| `/hooks` | Voir les configurations de hooks |
| `/ide` | Gérer les intégrations IDE |
| `/mcp` | Gérer les connexions aux serveurs MCP |
| `/plugin` | Gérer les plugins Claude Code |
| `/reload-plugins` | Recharger tous les plugins actifs sans redémarrer |
| `/reload-skills` | Rescanner les répertoires de skills sans redémarrer |
| `/skills` | Lister les skills disponibles |

**Memory (1)**

| Commande | Description |
|----------|-------------|
| `/memory` | Éditer les fichiers CLAUDE.md, activer/désactiver l'auto-mémoire |

**Model (7)**

| Commande | Description |
|----------|-------------|
| `/effort [low\|medium\|high\|xhigh\|max\|ultracode]` | Définir le niveau d'effort du modèle |
| `/fast [on\|off]` | Activer/désactiver le mode rapide |
| `/model [model]` | Sélectionner ou changer le modèle IA |
| `/passes` | Partager une semaine gratuite de Claude Code |
| `/plan [description]` | Entrer en mode plan depuis le prompt |
| `/ultraplan <prompt>` | Rédiger un plan dans une session ultraplan |

**Project (8)**

| Commande | Description |
|----------|-------------|
| `/add-dir <path>` | Ajouter un répertoire de travail pour la session |
| `/diff` | Ouvrir la visionneuse de diff interactive |
| `/init` | Initialiser le projet avec un guide CLAUDE.md |
| `/review` | Revue d'une pull request localement |
| `/security-review` | Analyser les changements en attente pour des vulnérabilités |
| `/team-onboarding` | Générer un guide d'onboarding d'équipe |
| `/ultrareview [PR]` | Revue de code profonde multi-agents dans un sandbox cloud |

**Remote (10)**

| Commande | Description |
|----------|-------------|
| `/autofix-pr [prompt]` | Spawner une session Claude Code web qui surveille la PR |
| `/desktop` | Continuer la session dans l'app Claude Code Desktop |
| `/install-github-app` | Configurer l'app GitHub Actions pour un dépôt |
| `/install-slack-app` | Installer l'app Claude Slack |
| `/mobile` | Afficher le QR code pour l'app mobile |
| `/remote-control` | Rendre cette session disponible pour contrôle à distance (alias: `/rc`) |
| `/remote-env` | Configurer l'environnement distant par défaut |
| `/schedule [description]` | Créer, mettre à jour, lister ou exécuter des routines (alias: `/routines`) |
| `/teleport` | Importer une session cloud dans ce terminal (alias: `/tp`) |
| `/web-setup` | Connecter votre compte GitHub à Claude Code sur le web |

**Session (14)**

| Commande | Description |
|----------|-------------|
| `/background [prompt]` | Détacher la session comme agent en arrière-plan (alias: `/bg`) |
| `/branch [name]` | Créer une branche de la conversation au point courant |
| `/btw <question>` | Poser une question rapide sans ajouter à la conversation |
| `/clear` | Démarrer une nouvelle conversation avec contexte vide |
| `/compact [instructions]` | Compacter la conversation avec instructions optionnelles |
| `/exit` | Quitter le CLI (alias: `/quit`) |
| `/fork <directive>` | Spawner un subagent forké qui hérite de la conversation complète |
| `/goal [condition\|clear]` | Définir un objectif — Claude continue jusqu'à ce qu'il soit atteint |
| `/recap` | Générer un résumé en une ligne de la session courante |
| `/rename [name]` | Renommer la session courante |
| `/resume [session]` | Reprendre une conversation par ID ou nom (alias: `/continue`) |
| `/rewind` | Revenir à un point précédent de la conversation |
| `/stop` | Arrêter la session en arrière-plan courante |
| `/workflows` | Ouvrir la vue de progression des workflows |

> **💡 Conseil :** Utilisez des slash commands pour chaque workflow "inner loop" que vous faites plusieurs fois par jour. Cela vous évite les prompts répétitifs et permet aussi à Claude d'utiliser ces workflows. Exemples typiques : `/commit-push-pr`, `/deploy`, `/run-tests`.

---

## 8. Quand utiliser Agent vs Command vs Skill ?

C'est l'une des questions les plus importantes pour bien architecturer votre configuration Claude Code. Voici une comparaison complète :

### Tableau de comparaison complet

| Dimension | Agent | Command | Skill |
|-----------|-------|---------|-------|
| **Emplacement** | `.claude/agents/<name>.md` | `.claude/commands/<name>.md` | `.claude/skills/<name>/SKILL.md` |
| **Contexte** | Processus subagent séparé | Inline (conversation principale) | Inline (conversation principale) |
| **Visible dans `/`** | Non | Oui — `/command-name` | Oui — `/skill-name` (sauf `user-invocable: false`) |
| **Auto-invoqué par Claude** | Oui — via champ `description` | Non | Oui — via champ `description` (sauf `disable-model-invocation: true`) |
| **Accepte des arguments** | Via paramètre `prompt` | `$ARGUMENTS`, `$0`, `$1` | `$ARGUMENTS`, `$0`, `$1` |
| **Injection de contexte dynamique** | Non | Oui — `` !`command` `` | Oui — `` !`command` `` |
| **Fenêtre de contexte propre** | Oui — toujours isolé | Non — partage le principal | Non — partage le principal (sauf `context: fork`) |
| **Override de modèle** | Champ `model:` | Champ `model:` | Champ `model:` |
| **Restrictions d'outils** | `tools:` / `disallowedTools:` | `allowed-tools:` | `allowed-tools:` |
| **Hooks** | Champ `hooks:` | — | Champ `hooks:` |
| **Mémoire persistante** | Champ `memory:` | — | — |
| **Peut précharger des skills** | Oui — champ `skills:` | — | — |
| **Serveurs MCP** | Champ `mcpServers:` | — | — |

### Règle de résolution

Quand plusieurs mécanismes peuvent satisfaire une même intention, Claude préfère **l'option la plus légère** :

```
1. Skill (inline, pas de surcharge de contexte)  ← préféré
2. Agent (contexte séparé, autonome)             ← utilisé si skill indisponible ou tâche complexe
3. Command (jamais — nécessite un / explicite)   ← uniquement si l'utilisateur tape /ma-commande
```

### Utilisez un Agent quand :

- La tâche est **autonome et multi-étapes** — l'agent doit explorer, décider et agir sans guidance constante
- Vous avez besoin d'**isolation du contexte** — le travail ne doit pas polluer la fenêtre de conversation principale
- L'agent a besoin d'une **mémoire persistante** entre les sessions
- Vous voulez **précharger des connaissances de domaine** via des skills sans encombrer le contexte principal
- La tâche bénéficie d'une **exécution en arrière-plan** ou dans un **git worktree**
- Vous avez besoin de **restrictions d'outils** ou d'un **mode de permission différent**

### Utilisez une Command quand :

- Vous avez besoin d'un **point d'entrée initié par l'utilisateur** — un workflow que l'utilisateur déclenche explicitement
- Le workflow implique d'**orchestrer** d'autres agents ou skills
- Vous voulez **garder le contexte lean** — le contenu de la commande n'est pas injecté dans le contexte de session tant que l'utilisateur ne la déclenche pas

### Utilisez un Skill quand :

- Vous voulez que **Claude auto-invoque** basé sur l'intention de l'utilisateur — les descriptions de skills sont injectées dans le contexte de la session
- La tâche est une **procédure réutilisable** qui peut être invoquée depuis plusieurs endroits
- Vous avez besoin de **préchargement dans un agent** — intégrer des connaissances de domaine dans un agent spécifique au démarrage

### Exemple concret : "Quelle heure est-il ?"

Ce dépôt a les trois mécanismes définis pour la même tâche — afficher l'heure en PKT. Voici ce qui se passe quand l'utilisateur tape "Quelle heure est-il ?" sans invoquer de commande explicite :

| Mécanisme | Se déclenche-t-il ? | Pourquoi / Pourquoi pas |
|-----------|---------------------|------------------------|
| `time-command` | Non | Les commandes ne sont **jamais auto-invoquées**. L'utilisateur doit taper `/time-command`. |
| `time-agent` | Possible | La description de l'agent correspond à l'intention. Claude peut le spawner via l'outil Agent. Mais c'est plus lourd qu'un skill. |
| `time-skill` | Très probable | La description du skill correspond. Claude l'invoque via l'outil Skill. Plus efficace car inline. |

> **💡 Conseil :** Si `disable-model-invocation: true` est défini sur le skill, Claude ne peut pas l'auto-invoquer. L'agent devient alors la seule option auto-invocable — au coût d'une fenêtre de contexte séparée pour une tâche triviale.

---

## 9. Architecture d'orchestration — Command → Agent → Skill

C'est le pattern d'architecture central de ce dépôt — la démonstration concrète de comment combiner les trois mécanismes pour construire des workflows complexes et maintenables.

### Le pattern en 3 couches

```
┌──────────────────────────────────────────────────────────────────┐
│              WORKFLOW D'ORCHESTRATION                            │
│           Command  →  Agent  →  Skill                            │
└──────────────────────────────────────────────────────────────────┘

                       ┌─────────────────────┐
                       │  Interaction User   │
                       └──────────┬──────────┘
                                  │
                                  ▼
       ┌──────────────────────────────────────────────────────┐
       │  /weather-orchestrator — Command (Point d'entrée)    │
       └──────────────────────────┬───────────────────────────┘
                                  │
                             Étape 1
                                  │
                                  ▼
                     ┌────────────────────────┐
                     │  AskUser — C° ou F° ?  │
                     └────────────┬───────────┘
                                  │
                        Étape 2 — outil Agent
                                  │
                                  ▼
       ┌──────────────────────────────────────────────────────┐
       │  weather-agent — Agent ● skill préchargé: weather-fetcher │
       └──────────────────────────┬───────────────────────────┘
                                  │
                         Retourne: temp + unité
                                  │
                        Étape 3 — outil Skill
                                  │
                                  ▼
       ┌──────────────────────────────────────────────────────┐
       │  weather-svg-creator — Skill ● carte SVG + output   │
       └──────────────────────────┬───────────────────────────┘
                                  │
                         ┌────────┴────────┐
                         │                 │
                         ▼                 ▼
                  ┌────────────┐    ┌────────────┐
                  │weather.svg │    │ output.md  │
                  └────────────┘    └────────────┘
```

### Deux patterns de skills dans un seul workflow

Le système météo démontre les deux patterns :

- **Agent Skills (préchargés)** : `weather-fetcher` est injecté dans `weather-agent` au démarrage comme connaissance de domaine
- **Skills (indépendants)** : `weather-svg-creator` est invoqué directement par la commande via l'outil Skill

### Code complet : weather-orchestrator (Command)

**Fichier** : `.claude/commands/weather-orchestrator.md`

```yaml
---
description: Fetch Dubai weather and create an SVG weather card
model: haiku
allowed-tools:
  - AskUserQuestion
  - Agent
  - Skill
---

# Weather Orchestrator Command

Fetch the current temperature for Dubai, UAE and create a visual SVG weather card.

## Execution Contract (non-negotiable)

You MUST complete this command by delegating to the `weather-agent` subagent. You are forbidden from:

- Fetching weather data yourself via Bash, WebFetch, or any other tool
- Skipping Step 1 (the user's unit preference is required input to the agent)
- Calling `weather-svg-creator` before the agent returns a temperature

If you cannot invoke the Agent tool, stop and report the error to the user. Do not improvise.

## Workflow

### Step 1: Ask User Preference

Use the AskUserQuestion tool to ask the user whether they want the temperature in Celsius or Fahrenheit.

### Step 2: Fetch Weather Data via Agent

Use the Agent tool to invoke the weather agent:

- subagent_type: weather-agent
- description: Fetch Dubai weather data
- prompt: Fetch the current temperature for Dubai, UAE in [unit requested by user].
  Return the numeric temperature value and unit.
- model: haiku

Wait for the agent to complete and capture the returned temperature value and unit.

**Fail-closed guardrail**: If the agent does not return a numeric temperature and unit,
DO NOT proceed to Step 3. Report the failure to the user and stop.

### Step 3: Create SVG Weather Card

Use the Skill tool to invoke the weather-svg-creator skill:

- skill: weather-svg-creator

## Output Summary

Provide a clear summary showing:
- Temperature unit requested
- Temperature fetched from Dubai
- SVG card created at `orchestration-workflow/weather.svg`
- Summary written to `orchestration-workflow/output.md`
```

### Code complet : weather-agent (Agent)

**Fichier** : `.claude/agents/weather-agent.md`

```yaml
---
name: weather-agent
description: Use this agent PROACTIVELY when you need to fetch weather data for Dubai, UAE.
  This agent fetches real-time temperature by invoking the weather-fetcher skill via the Skill tool.
allowedTools:
  - "Read"
  - "Skill"
model: sonnet
color: green
maxTurns: 5
permissionMode: acceptEdits
memory: project
skills:
  - weather-fetcher
hooks:
  PreToolUse:
    - matcher: ".*"
      hooks:
        - type: command
          command: python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py --agent=voice-hook-agent
          timeout: 5000
          async: true
---

# Weather Agent

You are a specialized weather agent that fetches weather data for Dubai, UAE.

## Execution Contract (non-negotiable)

You MUST fetch the temperature by invoking the `weather-fetcher` skill via the **Skill tool**.
You are forbidden from:

- Calling `WebFetch`, `WebSearch`, `curl`, or any HTTP/API tool yourself
- Reading the skill's instructions and executing them inline
- Skipping the Skill tool invocation for any reason

Your tool allowlist intentionally excludes network tools.

## Your Task

1. **Invoke**: Call the Skill tool with `skill: weather-fetcher`
2. **Report**: Return the temperature value and unit to the caller
3. **Memory**: Update your agent memory with the reading details for historical tracking

## Workflow

### Step 1: Invoke weather-fetcher skill

```
Skill(skill: "weather-fetcher")
```

### Step 2: Final Report

After the skill returns, provide a concise report to the caller:
- Temperature value (numeric)
- Temperature unit (Celsius or Fahrenheit)
- Comparison with previous reading (if available in memory)
```

### Code complet : weather-fetcher (Agent Skill)

**Fichier** : `.claude/skills/weather-fetcher/SKILL.md`

```yaml
---
name: weather-fetcher
description: Instructions for fetching current weather temperature data for Dubai, UAE from Open-Meteo API
user-invocable: false
allowed-tools:
  - "WebFetch(*)"
---

# Weather Fetcher Skill

## Task

Fetch the current temperature for Dubai, UAE in the requested unit (Celsius or Fahrenheit).

## Instructions

1. **Fetch Weather Data**: Use the WebFetch tool to get current weather data from Open-Meteo.

   For **Celsius**:
   - URL: `https://api.open-meteo.com/v1/forecast?latitude=25.2048&longitude=55.2708&current=temperature_2m&temperature_unit=celsius`

   For **Fahrenheit**:
   - URL: `https://api.open-meteo.com/v1/forecast?latitude=25.2048&longitude=55.2708&current=temperature_2m&temperature_unit=fahrenheit`

2. **Extract Temperature**: From the JSON response, extract `current.temperature_2m`

3. **Return Result**: Return the temperature value and unit clearly.

## Notes

- Open-Meteo is free, requires no API key, uses coordinate-based lookups
- Dubai coordinates: latitude 25.2048, longitude 55.2708
- Return ONLY the temperature — do not write files or transform the data
```

### Code complet : weather-svg-creator (Skill)

**Fichier** : `.claude/skills/weather-svg-creator/SKILL.md`

```yaml
---
name: weather-svg-creator
description: Creates an SVG weather card showing the current temperature for Dubai.
  Writes the SVG to orchestration-workflow/weather.svg and updates orchestration-workflow/output.md.
---

# Weather SVG Creator Skill

Creates a visual SVG weather card for Dubai, UAE and writes the output files.

## Task

You will receive a temperature value and unit (Celsius or Fahrenheit) from the calling context.
Create an SVG weather card and write both the SVG and a markdown summary.

## Instructions

1. **Create SVG** — Use the SVG template from reference.md, replacing placeholders with actual values
2. **Write SVG file** — Read then write to `orchestration-workflow/weather.svg`
3. **Write summary** — Read then write to `orchestration-workflow/output.md`

## Rules

- Use the exact temperature value and unit provided — do not re-fetch or modify
- The SVG must be self-contained and valid
- Both output files go in the `orchestration-workflow/` directory
```

### Pourquoi cette architecture — séparation des responsabilités

L'architecture en 3 couches apporte des bénéfices concrets :

1. **Command = orchestrateur** : gère l'interaction utilisateur et coordonne le workflow. Elle ne touche pas les APIs, ne crée pas de fichiers.
2. **Agent = collecteur de données** : travaille en contexte isolé, utilise ses skills préchargés, retourne un résultat propre.
3. **Skill = créateur de sortie** : procédure spécialisée et réutilisable. Elle peut être invoquée depuis n'importe quelle autre commande ou agent.

Cette séparation rend chaque composant testable, réutilisable et maintenable indépendamment.

> **💡 Conseil :** La commande sert de "fail-closed guardrail" — si l'agent ne retourne pas une température valide, elle s'arrête et rapporte l'erreur. Ne jamais improviser ou essayer de récupérer un résultat incorrect. Cette discipline est ce qui distingue un workflow robuste d'un workflow fragile.

---

## 10. Les Hooks

Les hooks sont des scripts exécutés à des moments précis du cycle de vie de Claude Code. Ils permettent d'automatiser des comportements déterministes — formatage du code, logging, notifications sonores, routage des permissions vers Slack, et bien plus.

### Concept fondamental

Un hook est un script externe (Python, bash, etc.) que Claude Code invoque à des événements définis. Contrairement à Claude lui-même, les hooks sont déterministes — ils s'exécutent toujours, indépendamment du raisonnement du modèle. C'est la couche d'automatisation fiable au-dessus de l'IA.

### Événements supportés

| Événement | Description |
|-----------|-------------|
| `PreToolUse` | Avant l'utilisation de tout outil |
| `PostToolUse` | Après l'utilisation réussie d'un outil |
| `PostToolUseFailure` | Après l'échec d'une utilisation d'outil |
| `PermissionRequest` | Quand Claude demande une permission |
| `PermissionDenied` | Quand une permission est refusée |
| `UserPromptSubmit` | Quand l'utilisateur soumet un prompt |
| `SessionStart` | Au démarrage d'une session |
| `SessionEnd` | À la fin d'une session |
| `Stop` | Quand Claude termine une réponse |
| `StopFailure` | Quand Claude s'arrête avec une erreur |
| `SubagentStart` | Quand un subagent commence |
| `SubagentStop` | Quand un subagent se termine |
| `PreCompact` | Avant la compaction du contexte |
| `PostCompact` | Après la compaction du contexte |
| `Setup` | Pendant la configuration initiale |
| `TeammateIdle` | Quand un coéquipier agent est inactif |
| `TaskCreated` | Quand une tâche est créée |
| `TaskCompleted` | Quand une tâche est complétée |
| `ConfigChange` | Quand la configuration change |
| `WorktreeCreate` | Quand un worktree est créé |
| `WorktreeRemove` | Quand un worktree est supprimé |
| `InstructionsLoaded` | Quand les instructions sont chargées |
| `FileChanged` | Quand un fichier surveillé change |
| `CwdChanged` | Quand le répertoire courant change |
| `Elicitation` | Lors d'une demande d'input |
| `ElicitationResult` | Quand une demande d'input reçoit une réponse |

### Configuration dans settings.json

Les hooks sont configurés dans `.claude/settings.json` sous la clé `hooks` :

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bun run format || true",
            "timeout": 5000,
            "async": false
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 .claude/hooks/scripts/notify.py",
            "timeout": 5000,
            "async": true,
            "once": true
          }
        ]
      }
    ]
  }
}
```

Le champ `matcher` est une expression régulière appliquée au nom de l'outil. `".*"` correspond à tous les outils. `"Write|Edit"` ne correspond qu'aux outils d'écriture.

### Pattern de handler Python unifié

Ce dépôt utilise un seul script Python `hooks.py` qui dispatch tous les événements. C'est une approche centralisée élégante :

```python
# .claude/hooks/scripts/hooks.py
import sys
import json
import os

event = os.environ.get("CLAUDE_HOOK_EVENT", "")
tool = os.environ.get("CLAUDE_TOOL_NAME", "")

# Dispatch basé sur l'événement
if event == "PreToolUse" and tool.startswith("Bash"):
    # Jouer un son spécifique pour les commits git
    if "git commit" in os.environ.get("CLAUDE_TOOL_INPUT", ""):
        play_sound("pretooluse-git-committing")
    else:
        play_sound("pretooluse")
elif event == "Stop":
    play_sound("stop")
elif event == "SubagentStart":
    play_sound("subagent-start")
```

### Hooks scoped aux agents

Les agents peuvent avoir leurs propres hooks, actifs uniquement pendant leur exécution :

```yaml
---
name: mon-agent
hooks:
  PreToolUse:
    - matcher: ".*"
      hooks:
        - type: command
          command: python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py --agent=mon-agent
          timeout: 5000
          async: true
  Stop:
    - hooks:
        - type: command
          command: echo "Agent terminé" | notify-send
---
```

### Hooks scoped aux skills

De même, les skills peuvent avoir des hooks qui ne s'activent que pendant l'exécution du skill :

```yaml
---
name: careful
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: python3 .claude/hooks/block-dangerous.py
---
```

Exemple pratique : `/careful` — bloque `rm -rf`, `DROP TABLE`, force-push, `kubectl delete` via un hook PreToolUse actif uniquement pendant la session.

### Désactiver les hooks

```json
// Dans .claude/settings.local.json
{
  "disableAllHooks": true
}
```

Pour désactiver des hooks individuels, modifiez `hooks-config.json` (partagé) et `hooks-config.local.json` (personnel, git-ignoré).

### Exemples pratiques

**Auto-formatage du code après chaque Write/Edit :**

```json
"PostToolUse": [{
  "matcher": "Write|Edit",
  "hooks": [{"type": "command", "command": "bun run format || true"}]
}]
```

**Routage des permissions vers Slack :**

```json
"PermissionRequest": [{
  "hooks": [{"type": "command", "command": "python3 .claude/hooks/slack-permission.py"}]
}]
```

**Notification de fin de tâche :**

```json
"Stop": [{
  "hooks": [{"type": "command", "command": "osascript -e 'display notification \"Claude terminé\" with title \"Claude Code\"'", "once": true}]
}]
```

> **💡 Conseil :** Utilisez des hooks pour automatiser la vérification du travail de Claude. Un hook `Stop` qui lance les tests unitaires et réinjecte les résultats dans la conversation force Claude à corriger ses erreurs avant de vous rendre la main.

> **💡 Conseil :** Le flag `once: true` sur un hook fait qu'il ne s'exécute qu'une seule fois par session, même si l'événement se déclenche plusieurs fois. Idéal pour les notifications de démarrage/fin de session.

---

## 11. Le Protocole MCP (Model Context Protocol)

Le MCP (Model Context Protocol) est un standard ouvert qui permet à Claude Code de se connecter à des outils, bases de données, APIs et services externes. Là où les tools natifs de Claude Code (Read, Write, Bash) couvrent les opérations système, les serveurs MCP étendent Claude à des domaines spécialisés.

### Concept fondamental

Un serveur MCP est un processus externe qui expose des outils à Claude via un protocole standardisé. Du point de vue de Claude, les outils MCP apparaissent comme des outils natifs, avec la convention de nommage `mcp__<server>__<tool>`.

### Configuration

Les serveurs MCP sont configurés dans `.mcp.json` à la racine du projet (portée projet) ou dans `~/.claude.json` (portée utilisateur).

**Exemple `.mcp.json` :**

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    },
    "deepwiki": {
      "command": "npx",
      "args": ["-y", "deepwiki-mcp"]
    },
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp"
    }
  }
}
```

Utilisez l'expansion de variables d'environnement pour les secrets au lieu de committer des clés API :

```json
{
  "mcpServers": {
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp?token=${MCP_API_TOKEN}"
    }
  }
}
```

### Types de serveurs

| Type | Transport | Exemple |
|------|-----------|---------|
| **stdio** | Spawne un processus local | `npx`, `python`, binaire |
| **http** | Se connecte à une URL distante | endpoint HTTP/SSE |

### Les 5 serveurs MCP recommandés pour le quotidien

| Serveur MCP | Ce qu'il fait | Source |
|-------------|--------------|--------|
| **Context7** | Récupère la documentation à jour des bibliothèques dans le contexte. Prévient les hallucinations d'APIs obsolètes | "De loin le meilleur MCP pour le code" — r/mcp |
| **Playwright** | Automatisation du navigateur — implémente, teste et vérifie les fonctionnalités UI de façon autonome. Screenshots, navigation, tests de formulaires | Essentiel pour le frontend |
| **Claude in Chrome** | Connecte Claude à votre vrai navigateur Chrome — inspecte la console, le réseau, le DOM. Débogue ce que les utilisateurs voient réellement | "Game changer" pour le débogage |
| **DeepWiki** | Récupère la documentation structurée de style wiki pour n'importe quel dépôt GitHub — architecture, surface API, relations | Idéal pour explorer des dépôts inconnus |
| **Excalidraw** | Génère des diagrammes d'architecture, flowcharts et designs système en sketches Excalidraw à partir de prompts | Parfait pour la documentation visuelle |

La chaîne de valeur : **Research** (Context7/DeepWiki) → **Debug** (Playwright/Chrome) → **Document** (Excalidraw).

### Portées des serveurs MCP

| Portée | Emplacement | Usage |
|--------|-------------|-------|
| **Projet** | `.mcp.json` (racine du dépôt) | Serveurs partagés par l'équipe, committés dans git |
| **Utilisateur** | `~/.claude.json` (clé `mcpServers`) | Serveurs personnels pour tous les projets |
| **Subagent** | Frontmatter de l'agent (champ `mcpServers`) | Serveurs scoped à un subagent spécifique |

Priorité : Subagent > Projet > Utilisateur.

### Paramètres de configuration MCP dans settings.json

| Clé | Type | Description |
|-----|------|-------------|
| `enableAllProjectMcpServers` | boolean | Auto-approuver tous les serveurs `.mcp.json` sans confirmation |
| `enabledMcpjsonServers` | array | Liste blanche de noms de serveurs à auto-approuver |
| `disabledMcpjsonServers` | array | Liste noire de noms de serveurs à rejeter |

### Règles de permissions pour les outils MCP

```json
{
  "permissions": {
    "allow": [
      "mcp__*",
      "mcp__context7__*",
      "mcp__playwright__browser_snapshot"
    ],
    "deny": [
      "mcp__dangerous-server__*"
    ]
  }
}
```

### MCP par subagent

Un agent peut avoir ses propres serveurs MCP, actifs uniquement pendant son exécution :

```yaml
---
name: web-researcher
mcpServers:
  - context7
  - deepwiki
tools: Read, Write
---
```

> **💡 Conseil :** Moins c'est plus avec les MCP. Un utilisateur Reddit avait installé 15 serveurs MCP pensant que "plus = mieux". Il s'est retrouvé à n'en utiliser que 4 quotidiennement. Commencez avec Context7 et Playwright — ils couvrent la grande majorité des besoins.

> **⚠️ Attention :** Ne jamais committer des clés API directement dans `.mcp.json`. Utilisez des variables d'environnement (`${MON_TOKEN}`) et assurez-vous que votre `.env` est git-ignoré.

---

## 12. Gestion du contexte

La fenêtre de contexte est la ressource la plus précieuse dans une session Claude Code. Comprendre ses limites et apprendre à la gérer est essentiel pour maintenir une haute qualité de travail sur des sessions longues.

### La "dumb zone" à 40%

Au-delà de 40% d'utilisation du contexte, la qualité des réponses commence à dégrader subtilement. Claude peut oublier des instructions données en début de session, faire des erreurs de cohérence, ou répéter du travail déjà fait. C'est la "zone de flou".

### La pourriture du contexte à 300-400K tokens

Au-delà de 300 000 à 400 000 tokens, même les meilleurs modèles commencent à avoir des problèmes de cohérence significatifs. C'est ce qu'on appelle le "context rot" — la dégradation progressive de la qualité liée à un contexte surchargé.

### `/compact` — compaction manuelle et automatique

`/compact [instructions]` résume la conversation tout en conservant les informations essentielles :

```
/compact focus sur les changements de code et les décisions d'architecture
```

La compaction automatique se déclenche à 80% d'utilisation par défaut (configurable via `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`).

### `/context` — visualisation

Exécutez `/context` pour voir une grille colorée de l'utilisation du contexte courant, avec des suggestions d'optimisation pour les outils lourds en contexte, les bloats de mémoire, et des avertissements de capacité.

### `/clear` — démarrage frais

`/clear` démarre une nouvelle conversation avec un contexte vide. La conversation précédente reste disponible dans `/resume`. À différencier de `/compact` qui garde le fil de la conversation mais le compresse.

### Stratégies de gestion du contexte

1. **Petites sessions focalisées** — traitez un problème à la fois, créez une nouvelle session pour les nouveaux sujets
2. **`/compact` à 50%** — proactif, bien avant d'atteindre la zone de flou
3. **Commits séparés par fichier** — ne bundlez pas tous les changements dans un seul commit. Chaque fichier a son commit avec un message descriptif
4. **`/clear` pour les nouveaux sujets** — ne continuez pas une session sur un sujet complètement différent
5. **Mode plan d'abord** — en mode plan, Claude n'exécute pas de code, donc utilise moins de contexte pour établir la stratégie
6. **Agents pour les tâches longues** — les agents ont leur propre contexte isolé, donc n'impactent pas votre session principale

> **⚠️ Attention :** Ne pas confondre `/compact` et `/clear`. `/compact` compresse l'historique mais continue la même conversation — utile pour libérer du contexte tout en gardant le fil. `/clear` efface tout et démarre neuf — utile quand vous changez complètement de sujet.

> **💡 Conseil :** Boris Cherny utilise `--bare` pour les invocations SDK non-interactives, ce qui accélère le démarrage jusqu'à 10x : `claude -p "résume ce codebase" --bare`. Par défaut, Claude cherche les CLAUDE.md, paramètres et MCP locaux — avec `--bare`, vous spécifiez explicitement ce que vous voulez charger.

---

## 13. Fonctionnalités avancées

### Git Worktrees — isolation complète

Les git worktrees permettent à plusieurs instances de Claude de travailler en parallèle sur le même dépôt, chacune dans sa propre copie de travail :

```bash
# Démarrer une session dans un worktree
claude -w
claude --worktree
```

Dans les agents, utilisez `isolation: "worktree"` pour que l'agent s'exécute dans un worktree temporaire (nettoyé automatiquement s'il n'y a pas de changements) :

```yaml
---
name: feature-builder
isolation: worktree
---
```

Boris Cherny maintient **des dizaines de Claudes en parallèle** grâce aux worktrees. La commande `/batch` pousse cela à l'extrême — elle fan-out des centaines, voire des milliers d'agents worktree pour des migrations de code massives.

### Agent Teams — multi-terminal en parallèle

Les Agent Teams permettent à plusieurs sessions Claude Code indépendantes de coordonner leur travail via une liste de tâches partagée :

**Activation :**

```json
// Dans ~/.claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**Lancement :**

```bash
brew install tmux
tmux new -s dev
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 claude
```

Deux modes de coordination :
- **In-process** (défaut) : tous les coéquipiers dans votre terminal
- **Split panes** : chaque coéquipier dans son propre panneau tmux ou iTerm2

Contrairement aux subagents (forks de contexte dans une session), chaque membre d'équipe est une session Claude Code complète avec son propre CLAUDE.md, serveurs MCP et skills chargés.

### Tâches planifiées — `/loop` et `/schedule`

`/loop` programme une tâche récurrente avec une granularité de cron (minimum 1 minute, expiration automatique après 3 jours) :

```
/loop 5m /babysit
/loop 30m /slack-feedback
/loop 1h /pr-pruner
```

`/schedule` (alias `/routines`) crée des routines persistantes via une conversation guidée.

### `/goal` — objectif persistant

`/goal <condition>` maintient Claude en travail à travers les tours jusqu'à ce qu'une condition soit satisfaite :

```
/goal all tests pass and the PR is approved
/goal clear
```

Claude Code v2.1.139+ requis. La condition est évaluée par Haiku à chaque tour.

### Auto Mode — Shift+Tab

Le mode auto élimine les confirmations de permissions répétitives. Activez-le avec `Shift+Tab` ou `--permission-mode auto`. À utiliser après avoir pré-approuvé les outils courants via `/permissions`.

### `/ultraplan` et `/ultrareview`

- `/ultraplan <prompt>` — rédige un plan dans une session cloud dédiée, le révise dans le navigateur, puis l'exécute à distance ou le renvoie au terminal
- `/ultrareview [PR]` — revue de code profonde multi-agents dans un sandbox cloud. Produit une revue structurée avec des findings priorisés

### Voice Dictation — `/voice`

Boris Cherny code principalement par dictée vocale. Activez avec `/voice hold` (maintenir la barre espace pour parler) ou `/voice tap` (toucher pour parler/arrêter).

### Remote Control — `/remote-control` et `/teleport`

- `/remote-control` (alias `/rc`) — rend la session locale contrôlable depuis claude.ai ou l'app mobile
- `/teleport` (alias `/tp`) — importe une session cloud dans votre terminal local

Boris recommande d'activer "Enable Remote Control for all sessions" dans `/config` pour travailler depuis mobile en déplacement.

### Claude Code sur le Web (claude.ai/code)

La version web permet de lancer des sessions dans des environnements cloud, de gérer des PR, et d'exécuter des workflows de longue durée sans terminal ouvert. Utilisez `/autofix-pr` pour spawner une session web qui surveille et corrige automatiquement votre PR CI.

### GitHub Actions

```yaml
# .github/workflows/claude.yml
- name: Claude Code Review
  uses: anthropics/claude-code-action@beta
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

Configurez avec `/install-github-app` pour la revue de code automatique sur les PRs.

### `/btw` — questions latérales

Posez une question rapide sans interrompre la tâche en cours :

```
/btw comment épelle-t-on "déterministe" ?
```

La réponse s'affiche en overlay sans polluer la conversation.

### Checkpointing automatique

Claude Code suit automatiquement toutes les modifications de fichiers. Utilisez `/rewind` (alias `/checkpoint`, `/undo`) pour revenir à un point précédent et annuler les changements correspondants.

---

## 14. Conseils pratiques

### Prompting

> **💡 Conseil :** La chose la plus importante pour obtenir d'excellents résultats de Claude Code : **donnez à Claude un moyen de vérifier son travail**. Si Claude a cette boucle de retour, il multipliera la qualité du résultat final par 2 ou 3.

> **💡 Conseil :** Évitez les termes vagues comme "lancer" qui peuvent être mal interprétés. Soyez explicite : "Utilisez l'outil Agent pour invoquer weather-agent" plutôt que "lancez l'agent météo".

> **💡 Conseil :** Demandez à Claude de proposer ses propres objectifs `/goal` : "Basé sur ce que tu sais de moi et de ce qu'on a construit ensemble, quels sont les 3 `/goal` que nous pourrions exécuter maintenant pour les meilleurs résultats ?"

### Planification

> **💡 Conseil :** Démarrez la plupart des sessions en **mode plan** (Shift+Tab deux fois). Si l'objectif est d'écrire une PR, utilisez le mode plan et itérez avec Claude jusqu'à ce que le plan vous convienne. Puis passez en mode acceptation automatique — Claude peut souvent 1-shot l'implémentation à partir d'un bon plan.

> **💡 Conseil :** Utilisez `/ultraplan` pour les tâches architecturalement complexes — le plan est généré dans un sandbox dédié avec plus de contexte, révisé dans le navigateur, puis exécuté.

> **💡 Conseil :** Pour les tâches multi-étapes, utilisez un workflow de liste de tâches avec validation humaine à chaque étape plutôt qu'une exécution en une seule fois. Cela donne plus de contrôle et permet des corrections à mi-chemin.

> **💡 Conseil :** Demandez à Claude de rédiger lui-même le prompt `/goal` pour votre session : "Lis cette session et ce dépôt, analyse en profondeur l'intention et les objectifs, puis écris-moi le prompt `/goal` pour cela."

> **💡 Conseil :** Décomposez les sous-tâches en unités suffisamment petites pour se compléter en moins de 50% de la fenêtre de contexte.

### Gestion du contexte

> **💡 Conseil :** Faites un `/compact` manuel à environ 50% d'utilisation du contexte — avant d'entrer dans la zone de flou.

> **💡 Conseil :** Évitez les grandes boucles qui accumulent du contexte : préférez des agents isolés pour les tâches longues.

> **💡 Conseil :** Utilisez `/context` pour visualiser l'utilisation du contexte et recevoir des suggestions d'optimisation.

> **💡 Conseil :** Pour les invocations SDK non-interactives, utilisez `--bare` pour un démarrage jusqu'à 10x plus rapide : `claude -p "..." --bare --output-format=stream-json`.

> **💡 Conseil :** Quand vous commencez un sujet complètement différent, utilisez `/clear` pour repartir à zéro. Ne portez pas le poids d'une longue conversation précédente dans un nouveau contexte.

### Sessions

> **💡 Conseil :** Exécutez 5 Claudes en parallèle dans votre terminal. Numérotez vos onglets 1–5 et utilisez les notifications système pour savoir quand un Claude a besoin d'input.

> **💡 Conseil :** Utilisez `claude.ai/code` pour encore plus de parallélisme — 5 à 10 Claudes sur le web en parallèle avec vos sessions locales.

> **💡 Conseil :** Utilisez `/branch` pour créer une branche de conversation — testez une approche différente sans perdre l'historique principal.

> **💡 Conseil :** Activez "Enable Remote Control for all sessions" pour travailler depuis votre téléphone quand vous n'êtes pas à votre ordinateur.

### CLAUDE.md et Rules

> **💡 Conseil :** Gardez chaque fichier CLAUDE.md sous 200 lignes pour une adhérence fiable. Au-delà, la qualité décroît.

> **💡 Conseil :** Toute l'équipe doit contribuer au CLAUDE.md plusieurs fois par semaine. Chaque fois que Claude fait quelque chose d'incorrect, ajoutez une règle pour éviter que cela se reproduise.

> **💡 Conseil :** Taggez `@claude` dans les PR de vos collègues pour demander des mises à jour du CLAUDE.md dans le cadre de la PR — c'est de l'ingénierie composée.

> **💡 Conseil :** Utilisez `.claude/rules/*.md` avec frontmatter `paths:` pour des règles qui ne se chargent que pour des types de fichiers spécifiques. Par exemple, des règles GraphQL qui ne se chargent que quand vous éditez des fichiers `.graphql`.

> **💡 Conseil :** Mettez votre `settings.json` dans git pour que toute l'équipe bénéficie des mêmes configurations — autorisations pré-approuvées, style de sortie, attributions.

### Agents, Commands, Skills

> **💡 Conseil :** Créez des agents spécialisés avec des skills (progressive disclosure) plutôt que des agents généralistes. Un `code-reviewer` avec le skill `security-checklist` est plus efficace qu'un agent "tout-en-un".

> **💡 Conseil :** Les agents ne peuvent pas invoquer d'autres agents via des commandes bash. Utilisez l'outil `Agent(subagent_type="...", prompt="...")`.

> **💡 Conseil :** Pour les skills, construisez une section "Gotchas" à partir des points d'échec réels. C'est souvent le contenu le plus utile.

> **💡 Conseil :** Utilisez des slash commands pour chaque workflow "inner loop" que vous faites plusieurs fois par jour. Exemples : `/commit-push-pr`, `/deploy-staging`, `/run-all-tests`.

> **💡 Conseil :** Le champ `description` d'un skill ou d'un agent est une instruction pour le modèle, pas pour l'humain. Rédigez-le pour déclencher l'auto-invocation au bon moment.

> **💡 Conseil :** Utilisez `user-invocable: false` pour les skills de connaissance de fond (préchargés dans des agents) qui ne doivent pas apparaître dans le menu `/`.

### Hooks

> **💡 Conseil :** Utilisez un hook `PostToolUse` sur `Write|Edit` pour auto-formater le code. Claude génère généralement du code bien formaté, mais le hook gère les 10% restants et évite des erreurs de CI.

> **💡 Conseil :** Les hooks s'exécutent avec `async: true` pour ne pas bloquer Claude. Mettez `async: false` uniquement si le hook doit impérativement terminer avant que Claude continue.

> **💡 Conseil :** Utilisez les hooks scoped aux skills pour des comportements opinionés — `/careful` bloque les commandes destructrices uniquement quand vous le demandez explicitement.

### Git et PR

> **💡 Conseil :** Créez des commits séparés par fichier — ne bundlez pas plusieurs changements dans un seul commit. Chaque fichier mérite son propre commit avec un message descriptif. Cela facilite le revert sélectif et la revue de code.

> **💡 Conseil :** Utilisez `/review` pour une revue locale rapide, et `/ultrareview` pour une analyse profonde multi-agents en sandbox cloud.

> **💡 Conseil :** Configurez `/install-github-app` et taggez `@claude` dans vos PRs pour des revues automatiques à chaque push.

> **💡 Conseil :** Utilisez `attribution.commit` dans settings.json pour co-signer automatiquement les commits générés avec Claude : `"Co-Authored-By: Claude <noreply@anthropic.com>"`.

### Débogage

> **💡 Conseil :** Utilisez le Chrome extension (ou Claude in Chrome MCP) chaque fois que vous travaillez sur du code web. Donnez à Claude la capacité de voir la console et le réseau — il itérera jusqu'à ce que le résultat soit bon.

> **💡 Conseil :** Partagez des screenshots quand vous signalez des problèmes visuels — Claude Code peut lire les images directement.

> **💡 Conseil :** Pour les tâches de très longue durée, utilisez un Stop hook pour valider le travail de façon déterministe, ou le plugin `ralph-wiggum` pour des boucles auto-vérifiantes.

> **💡 Conseil :** Les commandes de terminal longue durée doivent être exécutées comme tâches de fond (`/background`) pour une meilleure visibilité des logs.

### Workflows quotidiens

> **💡 Conseil :** Expérimentez avec la combinaison `/loop` + skills : `/loop 5m /babysit` pour surveiller et corriger automatiquement des PRs, `/loop 30m /slack-feedback` pour répondre aux retours Slack toutes les 30 minutes.

> **💡 Conseil :** Utilisez `/btw` pour des questions rapides pendant que Claude travaille — vous obtenez une réponse immédiate sans polluer la conversation principale.

> **💡 Conseil :** Utilisez `/voice` pour dicter des tâches — Boris Cherny écrit la majorité de son code par dictée vocale.

---

## 15. Dépannage

### `/doctor` — diagnostics

Exécutez `/doctor` pour diagnostiquer et vérifier votre installation. Les résultats s'affichent avec des icônes de statut. Appuyez sur `f` pour que Claude corrige automatiquement les problèmes signalés.

### Problèmes courants et solutions

**Problème : Claude ne suit pas les instructions du CLAUDE.md**

Solutions :
- Vérifiez que le fichier ne dépasse pas 200 lignes
- Reformulez les règles de façon plus directe ("Ne jamais..." plutôt que "Essayez d'éviter...")
- Utilisez `/memory` pour vérifier quels fichiers de mémoire sont chargés
- Vérifiez le chemin — le fichier est-il au bon emplacement dans l'arborescence ?

**Problème : Un agent invoque des outils qu'il ne devrait pas avoir**

Solutions :
- Vérifiez le champ `tools:` dans le frontmatter de l'agent
- Si `tools:` est omis, l'agent hérite de tous les outils — spécifiez explicitement la liste blanche
- Utilisez `disallowedTools:` pour une liste noire explicite

**Problème : Un skill ne se déclenche pas automatiquement**

Solutions :
- Vérifiez que `disable-model-invocation: false` (ou absent)
- Améliorez le champ `description` — rédigez-le comme une instruction de déclenchement pour le modèle
- Vérifiez le champ `when_to_use` pour des phrases déclencheurs supplémentaires

**Problème : Un serveur MCP ne se connecte pas**

Solutions :
- Exécutez `/mcp` pour voir le statut des connexions
- Vérifiez que les dépendances sont installées (`npx -y @upstash/context7-mcp` installe automatiquement)
- Vérifiez les variables d'environnement pour les tokens requis
- Consultez les logs de Claude in Chrome ou Playwright pour les erreurs réseau

**Problème : Permissions refusées de façon répétitive**

Solutions :
- Utilisez `/permissions` pour pré-approuver les outils courants
- Committez les règles dans `.claude/settings.json` pour partager avec l'équipe
- Évitez `--dangerously-skip-permissions` — préférez des autorisations explicites

### Outils de débogage MCP

- **Claude in Chrome** : inspectez la console et le réseau du navigateur pour les problèmes MCP frontend
- **Playwright** : automatisez et testez les interactions UI pour vérifier le comportement réel
- **Chrome DevTools** : analysez les requêtes réseau en profondeur

### Logs des hooks

Les logs des hooks se trouvent dans :

```
.claude/hooks/logs/hooks-log.jsonl
```

Chaque entrée de log contient l'événement, l'outil, la durée et les éventuelles erreurs.

### Flags de débogage utiles

```bash
# Démarrage sans chargement des configs locales (10x plus rapide pour les scripts)
claude -p "..." --bare

# Résumer une session spécifique
claude --resume <session-id>

# Forker une session existante
claude --resume <session-id> --fork-session

# Démarrer avec un agent spécifique
claude --agent=mon-agent

# Ajouter des répertoires supplémentaires
claude --add-dir /chemin/autre-repo
```

> **💡 Conseil :** Exécutez `/insights` pour un rapport d'analyse de vos sessions Claude Code, incluant les domaines de travail, les patterns d'interaction et les points de friction.

---

## 16. Ressources

### Documentation officielle

- [Claude Code Docs](https://code.claude.com/docs) — documentation principale
- [Subagents](https://code.claude.com/docs/en/sub-agents) — création d'agents personnalisés
- [Skills](https://code.claude.com/docs/en/skills) — création de skills
- [Slash Commands](https://code.claude.com/docs/en/slash-commands) — référence des commandes
- [Hooks](https://code.claude.com/docs/en/hooks) — guide des hooks
- [MCP Servers](https://code.claude.com/docs/en/mcp) — configuration MCP
- [Memory](https://code.claude.com/docs/en/memory) — gestion de la mémoire
- [Settings](https://code.claude.com/docs/en/settings) — référence des paramètres
- [Git Worktrees](https://code.claude.com/docs/en/worktrees) — utilisation des worktrees
- [Agent Teams](https://code.claude.com/docs/en/agent-teams) — équipes d'agents
- [CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) — historique des versions

### Ce dépôt

- [claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) — dépôt de référence de ce guide
  - `best-practice/` — guides par fonctionnalité
  - `implementation/` — exemples concrets implémentés
  - `reports/` — analyses approfondies
  - `tips/` — conseils de Boris Cherny et Thariq
  - `orchestration-workflow/` — diagramme et code du workflow météo

### Dépôts de workflows communautaires

| Dépôt | ★ | Description |
|-------|---|-------------|
| [Superpowers](https://github.com/obra/superpowers) | 217k | Brainstorming → git worktrees → plans → subagents. 14 skills |
| [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) | 206k | `/ecc:plan` → TDD → code-review → deploy. 63 agents, 121 commands, 300+ skills |
| [Matt Pocock Skills](https://github.com/mattpocock/skills) | 117k | `/grill-me` → PRD → TDD → `/diagnose`. 29 skills |
| [Spec Kit](https://github.com/github/spec-kit) | 108k | Constitution → specify → clarify → plan → implement. 9 commands |
| [gstack](https://github.com/garrytan/gstack) | 107k | office-hours → spec → design → review → ship → retro. 61 skills |

### Conseils de Boris Cherny (Créateur de Claude Code)

- [13 Tips — Janvier 2026](tips/claude-boris-13-tips-03-jan-26.md) — Setup de base, plan mode, subagents, hooks
- [12 Tips — Février 2026](tips/claude-boris-12-tips-12-feb-26.md) — Terminal, plugins, permissions, status line, output styles
- [15 Tips — Mars 2026](tips/claude-boris-15-tips-30-mar-26.md) — Mobile, teleport, /loop, hooks, worktrees, /batch, /voice
- Boris Cherny sur X : [@bcherny](https://x.com/bcherny)

### Conseils de Thariq (Anthropic)

- [Comment Anthropic utilise les Skills — Mars 2026](tips/claude-thariq-tips-17-mar-26.md) — 9 types de skills, 9 bonnes pratiques, distribution, mesure
- Thariq sur X : [@trq212](https://x.com/trq212)

### Skills officiels d'Anthropic

- [anthropics/skills](https://github.com/anthropics/skills/tree/main/skills) — dépôt officiel des skills bundled et communautaires

---

*Guide rédigé le 5 juin 2026. Basé sur Claude Code v2.1.162.*

*Sources principales : [claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice), documentation officielle Anthropic, tips de Boris Cherny et Thariq.*
