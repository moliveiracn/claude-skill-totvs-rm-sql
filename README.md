# TOTVS RM SQL — Claude Code Skill

**Generate, validate, and review SQL queries for TOTVS RM (CorporeRM / Classis .Net) in academic contexts.** Ships with a schema lookup CLI, a query validator, and four production-tested query skeletons.

> **Buy on Gumroad:** https://moliveiracn.gumroad.com/l/totvs-rm-sql-claude-code
> Single-user license · No redistribution · Lifetime updates for the current major version

---

## The problem

TOTVS RM's schema is sprawling (hundreds of `S*` and `F*` tables), poorly documented, and full of landmines:

- `CODCOLIGADA` must be propagated on every JOIN — forget one and your numbers silently double.
- `IDPERLET` (int surrogate) vs `CODPERLET` (`'2025/1'` string) — use the wrong one in the wrong place, get empty results or Cartesian products.
- `SHABILITACAO` vs `SHABILITACAOFILIAL` vs `SCURSO` — three tables, three different things, one wrong JOIN and you report the wrong program.
- Financial chain must go `SCONTRATO → SPARCELA → SLAN → FLAN`. Skipping `SLAN` is a silent bug.
- Test records (`PPE.NOME LIKE '%TESTE%'`) inflate every report unless explicitly excluded.

LLMs hallucinate column names in this schema constantly. You end up rewriting their output by hand.

## What this skill does

When you ask Claude for a TOTVS RM query, cube, Planilha.Net dynamic table, or Visual Formula, the skill:

1. **Clarifies scope** before writing — presencial or EAD, which course levels, what fields you actually need.
2. **Identifies the context type** (Filtro, Cubo, Planilha.Net, Fórmula Visual, Maintenance) — each has different NOLOCK, parameter, and output requirements.
3. **Picks a skeleton** from one of four production templates (enrollment, financial, grades, maintenance/data-fix).
4. **Looks up schema on demand** via `schema_lookup.py` — no hallucinated columns. Supports compact mode, FK traversal, and common-field decodes (STATUSLAN etc.).
5. **Assembles the query** using canonical JOIN paths (student→person, period→course level, habilitação chain, financial bridge).
6. **Applies mandatory filters** — test exclusion, CODSTATUS, CODTIPOCURSO, PJ bolsa for financial queries.
7. **Validates** via `validate_query.py` before delivering. Warnings include missing CODCOLIGADA, wrong NOLOCK placement, risky top-level DISTINCT, audit fields missing on writes.

## What's in the box

```
totvs-rm-sql/
├── SKILL.md                           # The skill definition
├── schema_curated.jsonl               # Curated schema metadata (~1.3MB, 993 tables)
├── schema_lookup.py                   # CLI: lookup, search, FK, decode
├── validate_query.py                  # Query validator (report + maintenance modes)
└── references/
    ├── skeleton_enrollment.sql        # Student roster / matrícula reports
    ├── skeleton_financial.sql         # Boletos, payments, inadimplência
    ├── skeleton_grades.sql            # Notas, faltas, aprovação
    ├── skeleton_maintenance.sql       # UPDATE/DELETE generator + snapshot pattern
    ├── header.sql                     # Query header template
    ├── style_guide.md                 # Formatting, params, mandatory filters
    ├── cte_patterns.md                # Period CTE + ROW_NUMBER dedup
    └── join_paths.md                  # Canonical JOIN chains
```

## Trigger phrases

- "SQL query", "report", "filter", "cube", "Planilha.Net", "Fórmula Visual", "Visual Formula"
- "Filtro RM", "Cubo", "Planilha Dinâmica"
- "período letivo", "matrícula", "habilitação", "RA"
- Any TOTVS-specific table name (`SMATRICPL`, `SALUNO`, `SHISTORICO`, `SPLETIVO`, `SHABILITACAOFILIAL`, etc.)
- "review this RM query", "fix this RM query", "explain this RM query"

## Install (after purchase)

1. Unzip the package.
2. Copy the `totvs-rm-sql/` folder into your project's `.claude/skills/` directory (or your user-level skills directory):

```
your-project/
└── .claude/
    └── skills/
        └── totvs-rm-sql/
            ├── SKILL.md
            ├── schema_curated.jsonl
            ├── schema_lookup.py
            ├── validate_query.py
            └── references/
```

3. Requires Python 3 for `schema_lookup.py` and `validate_query.py` (both stdlib-only, no pip installs).
4. Restart or `/clear` your Claude Code session.

## Who this is for

Developers, analysts, and DBAs working with TOTVS RM at Brazilian universities (Classis .Net academic modules). If you're writing reports from `SMATRICPL`, `SALUNO`, or the Fluxus financial tables, this is for you.

If your organization uses TOTVS RM outside academic contexts (Healthcare, Logistics, etc.), the skeletons won't fit out of the box — but the schema lookup, validator, and style guide still apply.

## Limitations

- The curated schema reflects common Classis .Net tables. Customer-personalized `*COMPL` (campos complementares) tables are **not** included — those vary by install. Rare or custom tables may not be indexed; use `schema_lookup.py --search <name>` to check.
- Skeletons assume academic context (enrollment, grades, financial for tuition). Pure ERP / HR / logistics queries are out of scope.
- Maintenance / data-fix generators always use `BEGIN TRAN` wrapping per convention — you are responsible for inspecting output before committing.

## Requirements

- Claude Code
- Python 3 (stdlib only)
- Read access to a TOTVS RM database (SQL Server)

## License

Single-user, no redistribution. Full terms ship in `LICENSE.txt` inside the zip.
