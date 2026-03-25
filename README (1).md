# FLT3 Variant Pathogenicity Predictor

Predict whether an FLT3 nucleotide variant is pathogenic or benign from CDS and amino acid notation. For research use only.

## Deploying to Render (first time)

1. Create a new GitHub repository and push these files
2. Go to [render.com](https://render.com) → **New** → **Static Site**
3. Connect your GitHub repo
4. Set:
   - **Publish directory:** `.`
   - **Build command:** *(leave blank)*
5. Click **Deploy** — done

To update the live site after any code change: just push to GitHub. Render redeploys automatically.

---

## Adding new variant data

Open `index.html` and find:

```javascript
const KNOWN_VARIANTS = {
```

Add a new line inside that object:

```javascript
"c.XXXXN>N|p.RefPosAlt": { score: 0.XX, source: "Your source", mechanism: "What this mutation does." },
```

**Score guide:**
| Score | Classification |
|-------|---------------|
| 0.90 – 1.0 | Pathogenic |
| 0.75 – 0.89 | Likely Pathogenic |
| 0.45 – 0.74 | VUS |
| 0.20 – 0.44 | Likely Benign |
| 0.0 – 0.19 | Benign |

Then commit and push — Render redeploys automatically.

---

## Adjusting model weights (for novel variants)

When you have more data, you may want to adjust how much weight each feature carries. Find:

```javascript
const W = { domain: 0.42, substitution: 0.35, context: 0.23 };
```

- `domain` — weight for FLT3 domain/codon position
- `substitution` — weight for amino acid physicochemical severity (Grantham distance)
- `context` — weight for mutation type context

The three values must always sum to 1.0.

---

## What the model does

For **known variants**, it returns the score directly from the reference database (ClinVar, COSMIC, published AML cohorts).

For **novel variants**, it scores based on three features:
1. **Domain position** — where in FLT3 the mutation falls (activation loop scores highest)
2. **AA change severity** — Grantham physicochemical distance between reference and alternate amino acid (higher = more radical = more likely damaging)
3. **Mutation context** — type of mutation (ITD, nonsense, frameshift all score high; synonymous scores 0)

Hard guardrails apply first:
- Same nucleotide ref and alt (e.g. `c.300T>T`) → 2% (not a mutation)
- Synonymous amino acid change (e.g. `p.Ile836Ile`) → 3% (no protein change)
- ITD → 97% (always pathogenic)
- Nonsense / stop-gain → 91%
- Frameshift → 89%

---

## Data sources

- ClinVar (NLM/NCBI)
- COSMIC v98
- Mirza et al., *Int. J. Mol. Sci.* 2024
- Yamamoto et al., *Blood* 2001
- Smith et al., 2015
- Thiengtrong et al., 2022 (Thailand AML cohort)
- PMC7004512 (FLT3 inhibitor resistance mutations)

---

*For research use only. Not for clinical decision-making.*
