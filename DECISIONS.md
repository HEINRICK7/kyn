# Kyn Architecture Decision Record
Toda decisão deve ser registrada.

## Template

```
ADR-ID:

DATE:

STATUS:
PROPOSED / ACCEPTED / REJECTED / SUPERSEDED

CONTEXT:

DECISION:

ALTERNATIVES:

REASON:

EVIDENCE:

CONSEQUENCES:
```

## Regra
Sem evidência experimental:

```
EVIDENCE:
NOT_YET_MEASURED
```
Não inventar justificativas numéricas.

## ADR-001 — Toolchain inicial

```
ADR-ID: ADR-001
DATE: 2026-10-02
STATUS: ACCEPTED

CONTEXT:
M0 precisa de configuração de pacote e lockfile. O ambiente local disponibiliza Python 3.12.3 e uv.

DECISION:
Usar uv com uv.lock. Restringir inicialmente o pacote a Python >=3.12,<3.13. Resolver PyTorch pelo índice CPU oficial.

ALTERNATIVES:
requirements.lock; PyTorch do índice padrão.

REASON:
O toolchain segue o ambiente disponível e o requisito CPU-first. O índice CPU está documentado pelo guia oficial de integração uv/PyTorch: https://docs.astral.sh/uv/guides/integration/pytorch/.

EVIDENCE:
Ambiente local observado; configuração de dependências apenas. Evidência científica do Kyn: NOT_YET_MEASURED.

CONSEQUENCES:
Reavaliar a faixa de Python após validar o forward real em outros ambientes. O lockfile fixa dependências, mas não valida desempenho, compatibilidade do modelo nem resultados científicos.
```
