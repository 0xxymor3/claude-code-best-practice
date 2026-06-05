# Plan : Guide Complet de Claude Code en Français

## Contexte

L'utilisateur souhaite un guide complet de Claude Code rédigé en français, à partir du contenu de ce dépôt (`claude-code-best-practice`). Ce dépôt est la référence la plus à jour sur les bonnes pratiques Claude Code (122+ fichiers Markdown, 135 MB), maintenu par Shayan Raisshan avec des contributions directes de Boris Cherny (créateur de Claude Code) et de l'équipe Anthropic.

**Problème adressé** : Il n'existe pas de documentation complète de Claude Code en français. Les développeurs francophones doivent naviguer des dizaines de fichiers en anglais pour comprendre l'écosystème.

**Résultat attendu** : Un seul fichier Markdown complet, autonome, en français, qui couvre l'installation jusqu'aux patterns avancés d'orchestration.

---

## Format retenu

**Fichier unique** : `reports/guide-claude-code-francais.md`

Justification :
- Portable et partageable (un lien, un fichier)
- Compatible avec le pattern existant du dépôt (docs dans `reports/`)
- Consultable offline, searchable, versionnable
- Markdown avec tables des matières ancrées pour la navigation

**Longueur estimée** : ~2 000–2 500 lignes (guide de référence exhaustif)

**Conventions linguistiques** :
- Titres et explications en français
- Termes techniques conservés en anglais (`skill`, `hook`, `agent`, `command`, `subagent`, `MCP`) — standard en écriture technique française
- Code examples en anglais (non traduits)
- Callouts : `> **💡 Conseil :**` pour les tips, `> **⚠️ Attention :**` pour les pièges

---

## Structure du guide (outline complet)

```
# Guide Complet de Claude Code 🇫🇷

## Table des matières (avec ancres)

## 1. Qu'est-ce que Claude Code ?
   - Définition et positionnement
   - De l'ingénierie vibe à l'ingénierie agentique
   - Architecture d'ensemble

## 2. Installation et premiers pas
   - Prérequis (Node.js 18+)
   - Installation (npm/pip)
   - Authentification
   - Premier lancement
   - Structure du répertoire .claude/

## 3. La Mémoire — CLAUDE.md et Rules
   - Qu'est-ce que CLAUDE.md ?
   - Chargement ancestral vs descendant (monorepo)
   - .claude/rules/ — chargement conditionnel par paths
   - ~/.claude/CLAUDE.md — mémoire globale
   - Mémoire persistante des agents (user/project/local)
   - Bonnes pratiques d'écriture

## 4. Configuration — settings.json
   - Hiérarchie de configuration (5 niveaux)
   - Permissions (allow / ask / deny)
   - Modèle et effort
   - Style de sortie
   - Ligne de statut
   - Attribution des commits

## 5. Les Agents (Subagents)
   - Concept et cas d'usage
   - Structure d'un fichier agent (.claude/agents/<name>.md)
   - Les 16 champs du frontmatter
   - Les 5 agents officiels
   - Patterns : exécution contracts, learnings

## 6. Les Compétences (Skills)
   - Concept et deux modes d'invocation
   - Structure d'un fichier skill (.claude/skills/<name>/SKILL.md)
   - Les 16 champs du frontmatter
   - Les 10 skills officielles (code-review, batch, debug, loop…)
   - Agent skills vs Skills invoquées

## 7. Les Commandes (Slash Commands)
   - Concept : point d'entrée initié par l'utilisateur
   - Structure d'un fichier command (.claude/commands/<name>.md)
   - Les 16 champs du frontmatter
   - Les 83 commandes officielles (catégorisées)
   - Commandes personnalisées

## 8. Quand utiliser Agent vs Command vs Skill ?
   - Tableau comparatif complet
   - Règle de résolution (skill → agent → command)
   - Exemple concret : "Quelle heure est-il ?"

## 9. Architecture d'orchestration — Command → Agent → Skill
   - Le patron en 3 couches
   - Exemple complet : le système météo
   - Diagramme ASCII du flux
   - Code des 3 composants

## 10. Les Hooks
    - Concept et 17+ événements supportés
    - Configuration dans settings.json
    - Handler Python unifié
    - Hooks scoped (par agent, par skill)
    - Exemples pratiques

## 11. Le Protocole MCP (Model Context Protocol)
    - Qu'est-ce que MCP ?
    - Configuration (.mcp.json et settings.json)
    - 5 serveurs MCP recommandés
    - Serveurs MCP par subagent

## 12. Gestion du contexte
    - La "zone muette" (dumb zone) à 40%
    - Context rot à 300-400K tokens
    - /compact manuel et auto-compaction
    - Visualiser le contexte avec /context
    - Stratégies de préservation

## 13. Fonctionnalités avancées
    - Git Worktrees (isolation: "worktree")
    - Agent Teams (multi-terminaux)
    - Tâches planifiées (/loop, /schedule)
    - Mode Auto (Shift+Tab)
    - Ultraplan et Ultrareview
    - /goal — objectif persistant
    - Voice Dictation
    - Remote Control

## 14. Conseils pratiques (83+ tips)
    - Prompting
    - Planification
    - Sessions et contexte
    - CLAUDE.md et Rules
    - Agents, Commands, Skills
    - Hooks
    - Git et PR
    - Débogage
    - Workflows quotidiens

## 15. Dépannage
    - /doctor
    - Outils de débogage MCP (Playwright, Chrome DevTools)
    - Logs des hooks
    - Commandes utiles

## 16. Ressources
    - Documentation officielle
    - Dépôts de référence
    - Workflows communautaires
```

---

## Mapping sources → sections

| Section | Fichiers sources principaux |
|---------|---------------------------|
| 1. Introduction | `README.md` (CONCEPTS table), `tutorial/day1/README.md` |
| 2. Installation | `tutorial/day0/{mac,linux,windows}.md`, `README.md` |
| 3. Mémoire | `best-practice/claude-memory.md`, `reports/claude-agent-memory.md` |
| 4. Configuration | `best-practice/claude-settings.md`, `reports/claude-global-vs-project-settings.md`, `.claude/settings.json` |
| 5. Agents | `best-practice/claude-subagents.md`, `implementation/claude-subagents-implementation.md`, `.claude/agents/*.md` |
| 6. Skills | `best-practice/claude-skills.md`, `implementation/claude-skills-implementation.md`, `.claude/skills/*/SKILL.md` |
| 7. Commands | `best-practice/claude-commands.md`, `implementation/claude-commands-implementation.md` |
| 8. Comparaison | `reports/claude-agent-command-skill.md` |
| 9. Orchestration | `orchestration-workflow/orchestration-workflow.md`, `.claude/commands/weather-orchestrator.md`, `.claude/agents/weather-agent.md` |
| 10. Hooks | `.claude/settings.json`, `.claude/hooks/scripts/hooks.py`, `CLAUDE.md` |
| 11. MCP | `best-practice/claude-mcp.md`, `.mcp.json` |
| 12. Contexte | `tips/claude-boris-12-tips-12-feb-26.md`, `reports/llm-day-to-day-degradation.md` |
| 13. Avancé | `README.md` (Hot section), `implementation/claude-agent-teams-implementation.md`, `implementation/claude-scheduled-tasks-implementation.md`, `implementation/claude-goal-implementation.md` |
| 14. Conseils | `tips/claude-boris-*.md`, `tips/claude-thariq-*.md` (toutes catégories) |
| 15. Dépannage | `reports/claude-in-chrome-v-chrome-devtools-mcp.md`, `best-practice/claude-cli-startup-flags.md` |

---

## Fichiers à lire avant d'écrire

Ces fichiers doivent être lus intégralement pour extraire leur contenu dans le guide :

**Priorité haute** (contenu direct) :
- `best-practice/claude-memory.md` ✅ (déjà lu)
- `best-practice/claude-subagents.md` ✅ (déjà lu)
- `best-practice/claude-skills.md` ✅ (déjà lu)
- `best-practice/claude-commands.md` ✅ (déjà lu, partiel)
- `reports/claude-agent-command-skill.md` ✅ (déjà lu)
- `best-practice/claude-mcp.md`
- `reports/claude-agent-memory.md`
- `tutorial/day0/README.md`

**Priorité moyenne** (tips à sélectionner) :
- `tips/claude-boris-13-tips-03-jan-26.md`
- `tips/claude-boris-12-tips-12-feb-26.md`
- `tips/claude-thariq-tips-17-mar-26.md`

**Priorité basse** (pour les sections avancées) :
- `implementation/claude-agent-teams-implementation.md`
- `implementation/claude-scheduled-tasks-implementation.md`
- `reports/claude-global-vs-project-settings.md`

---

## Stratégie d'écriture

Le guide sera écrit en **une seule passe** par un agent général avec `bypassPermissions` pour accéder à tous les fichiers source. L'agent :

1. Lit tous les fichiers source prioritaires en parallèle
2. Synthétise le contenu section par section
3. Écrit le guide en français fluide avec exemples de code
4. Inclut des tableaux comparatifs tirés directement des sources
5. Ajoute les callouts de conseils (>💡) pour les meilleures pratiques
6. Inclut un diagramme ASCII de l'architecture d'orchestration

**Output** : `reports/guide-claude-code-francais.md`

---

## Vérification

Après écriture :
1. Vérifier que le fichier contient les 16 sections
2. Vérifier que tous les blocs de code sont correctement fermés
3. Vérifier que la table des matières correspond aux ancres réelles
4. Vérifier que les termes techniques en anglais sont cohérents
5. S'assurer que le fichier est autonome (pas de dépendances sur des images)
