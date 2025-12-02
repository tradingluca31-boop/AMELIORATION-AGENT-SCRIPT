# AMELIORATION-AGENT-SCRIPT

> **Prompts & Examples for Claude Code - RL Agent Debugging**

This repo contains **prompt templates** and **examples** to help other Claude Code users debug their RL trading agents.

---

## 0. BEHAVIORAL ANALYSIS - PSYCHANALYSE DE L'AGENT (NEW!)

**The most complete prompt** - Understand WHY your agent doesn't trade:

```
Je veux comprendre POURQUOI mon agent n'ouvre pas de positions ou très peu.
Crée un script de psychanalyse comportementale qui répond à ces questions:

Q1: DISTRIBUTION - Que choisit l'agent ? (SELL/HOLD/BUY %)
Q2: PEUR DU RISQUE - A-t-il peur d'ouvrir ? Préfère-t-il HOLD ?
Q3: FERMETURE - Quand il ouvre, ferme-t-il ses positions ?
Q4: REWARDS - Quels rewards reçoit-il pour chaque action ?
Q5: CONTEXTE - Dans quel contexte ouvre-t-il (si jamais) ?
Q6: OBSERVATIONS - Que "voit" l'agent ? Les features sont-elles informatives ?
Q7: ÉVOLUTION - Le comportement change-t-il au fil du temps ?
Q8: SYNTHÈSE - Quel est son "profil psychologique" ?

Je veux comprendre comment il travaille step by step.
Donne-moi un DIAGNOSTIC COMPORTEMENTAL avec hypothèses.
```

**Output Example:**

```
DIAGNOSTIC COMPORTEMENTAL:
  1. BLOQUÉ: N'ouvre jamais de positions
  2. MAL RÉCOMPENSÉ: HOLD plus profitable que TRADE

HYPOTHÈSE PRINCIPALE: L'agent est BLOQUÉ (ne peut pas ouvrir)
Causes possibles:
  1. FIX 8 (Over-Trading Protection) bloque les trades
  2. Bug dans _open_position()

RECOMMANDATION: Vérifier le code de _open_position()
```

---

## 1. Interview Prompt (Diagnostic)

Use this prompt when your agent has **0 trades** or unexpected behavior:

```
Je veux que tu fasses un interview pour savoir pourquoi il y'a 0 trades.
Pose les questions bien posées:
  - Q1: Les positions S'OUVRENT-elles ?
  - Q2: Le TP/SL est-il ATTEIGNABLE ?
  - Q3: Que se passe-t-il sur 100 steps random ?
  - Q4: Y a-t-il un mécanisme qui BLOQUE les trades ?
  - Q5: Simulation longue 500 steps

Crée un script Python qui teste tout ça automatiquement.
```

---

## 2. Auto-Critique Prompt

After each Claude response, add this to get self-evaluation:

```
Analyse cette réponse. Note-la sur 5 critères :
- Précision (1-10)
- Complétude (1-10)
- Pertinence (1-10)
- Clarté (1-10)
- Fiabilité des sources (1-10)

Justifie chaque note et propose une amélioration concrète.
```

**Example Output:**

| Critère | Note | Justification |
|---------|------|---------------|
| Précision | 9/10 | Bug correctly identified from diagnostic output |
| Complétude | 8/10 | Missed checking other similar variables |
| Pertinence | 10/10 | Fix directly addresses root cause |
| Clarté | 8/10 | Could add ASCII diagram |
| Fiabilité | 9/10 | Based on real code, not tested locally |

**Score: 44/50 (88%)**

---

## 3. Bug Hunting Prompt

When you suspect a bug but don't know where:

```
Cherche dans le code tous les endroits où [VARIABLE] est modifiée.
Vérifie si elle est correctement réinitialisée dans reset().
Montre-moi la ligne exacte du problème.
```

**Example:**
```
Cherche dans trading_env.py tous les endroits où last_trade_open_step est modifiée.
Vérifie si elle est correctement réinitialisée dans reset().
```

---

## 4. Fix Verification Prompt

After applying a fix:

```
Maintenant vérifie que le fix fonctionne:
1. Lis le code modifié
2. Explique pourquoi ça résout le problème
3. Propose un test rapide pour valider
4. Y a-t-il d'autres variables similaires à vérifier ?
```

---

## 5. Smoke Test Prompt

Quick validation after fixes:

```
Crée un smoke test de 10K steps qui:
- Track le nombre de positions OUVERTES
- Track le nombre de positions FERMÉES
- Affiche la distribution des actions (SELL/HOLD/BUY %)
- Donne un verdict SUCCESS/PARTIAL/FAILURE
```

---

## 6. Price Data Verification Prompt

When TP/SL never triggers:

```
Vérifie que les données de prix sont correctes:
- Quel est le prix dans prices_df ? (devrait être ~$1000-2000 pour Gold)
- Est-ce que c'est bien XAUUSD et pas EURUSD (~$1.5) ?
- Les colonnes OHLCV sont-elles présentes ?
```

---

## 7. Check Q-Values / Policy Output Prompt (NEW!)

See what your agent "thinks":

```
Crée un script qui montre ce que l'agent "pense" vraiment:

1. Affiche les probabilités d'action (policy output) pour PPO
   - P(SELL), P(HOLD), P(BUY)
   - Si P(HOLD) >> P(TRADE) → Agent a appris que ne rien faire est "safe"

2. Calcule l'entropy de la policy
   - Entropy basse (< 0.3) → Agent trop déterministe, n'explore plus
   - Entropy haute (> 0.8) → Bonne exploration

3. Détecte le mode collapse
   - Si une action > 70% → WARNING

Usage: python analysis/check_qvalues.py --model models/best_model.zip
```

---

## 8. Check Reward Function Prompt (NEW!)

Checklist for reward function problems:

```
Crée un script qui vérifie ma reward function avec ces 4 questions:

A) Est-ce que HOLD donne un reward (même petit)?
   → Si oui: l'agent est RÉCOMPENSÉ pour ne rien faire!

B) Les trades perdants sont-ils TROP pénalisés?
   → Si ratio |perte|/|gain| > 3: l'agent a PEUR de perdre

C) Y a-t-il un reward UNIQUEMENT à la clôture?
   → PIÈGE: Si reward qu'à clôture + pertes pénalisées
   → Agent apprend: ne jamais ouvrir = jamais de perte = SAFE

D) Le risk/reward est-il encouragé?
   → Gros gains doivent être mieux récompensés que petits gains

Donne-moi un verdict: PROBLEME / ATTENTION / OK pour chaque question.
```

---

## 9. Full Diagnostic Prompt (NEW!)

Complete diagnostic in one script:

```
Crée un diagnostic COMPLET qui fait 6 tests:

TEST 1: Distribution des actions (HOLD/BUY/SELL %)
TEST 2: Random vs Trained comparison (si model dispo)
TEST 3: Observations normalization check
TEST 4: Reward per action type
TEST 5: Exploration (entropy) analysis
TEST 6: Positions analysis

À la fin, donne:
- Liste des PROBLÈMES détectés
- Liste des SOLUTIONS recommandées
- VERDICT: AGENT NECESSITE REFONTE / A CORRIGER / SEMBLE OK

Usage: python analysis/full_diagnostic.py --episodes 20
```

---

## 10. Full Diagnostic Workflow

Complete prompt sequence for debugging 0 trades:

```
# Step 1 - Initial Interview
"Mon agent RL a 0 trades après 10K steps. Fais un diagnostic complet."

# Step 2 - After diagnostic
"Tu as trouvé le problème. Applique le fix et explique pourquoi ça marche."

# Step 3 - Auto-critique
"Analyse ta réponse sur 5 critères (Précision, Complétude, Pertinence, Clarté, Fiabilité). Note /10 chaque critère."

# Step 4 - Verification
"Push le fix sur GitHub et crée un test pour vérifier."

# Step 5 - Smoke test
"Lance un smoke test 10K steps pour confirmer que l'agent trade maintenant."
```

---

## Bugs Found Using These Prompts

| Bug | Prompt Used | Fix |
|-----|-------------|-----|
| Early returns in reward | Interview Q3 | Changed `return X` to `reward += X` |
| Wrong price data | Price verification | Use `auxiliary_data['xauusd_raw']['H1']` |
| FIX 8 blocking all trades | Interview Q4 | Reset `last_trade_open_step` in `reset()` |

---

## Links

| Repo | Description |
|------|-------------|
| [AGENT-8-UNIQUEMENT-](https://github.com/tradingluca31-boop/AGENT-8-UNIQUEMENT-) | Agent 8 centralized code |
| [AMELIORATION-AGENT-SCRIPT](https://github.com/tradingluca31-boop/AMELIORATION-AGENT-SCRIPT) | This repo - Prompts & examples |

---

**Last Updated**: 2025-12-02
**Author**: Trading Luca
**Purpose**: Help other Claude Code users debug their RL agents