# Kyn
Kyn é um sistema de decisão não-autoregressivo, CPU-first, baseado inicialmente no `LFM2.5-Encoder-230M`.

## Arquivos obrigatórios

A documentação operacional do projeto deve ser tratada como contrato de integridade. Os arquivos obrigatórios incluem:

1. [AGENT_RULES.md](AGENT_RULES.md)
2. [SCIENTIFIC_INTEGRITY.md](SCIENTIFIC_INTEGRITY.md)
3. [ARCHITECTURE.md](ARCHITECTURE.md)
4. [DATA_SPEC.md](DATA_SPEC.md)
5. [TRAINING_SPEC.md](TRAINING_SPEC.md)
6. [CPU_SPEC.md](CPU_SPEC.md)
7. [EXPERIMENT_PROTOCOL.md](EXPERIMENT_PROTOCOL.md)
8. [TESTING_STRATEGY.md](TESTING_STRATEGY.md)
9. [METRICS_SPEC.md](METRICS_SPEC.md)
10. [ROADMAP.md](ROADMAP.md)
11. [DECISIONS.md](DECISIONS.md)
12. [KNOWN_LIMITATIONS.md](KNOWN_LIMITATIONS.md)

O sistema recebe:

```
state
question
candidates[]
```
e produz:

```
scores
probabilities
uncertainty
DECIDE ou ABSTAIN
```

## Objetivo
Construir e validar cientificamente um modelo especializado em decisão conjunta entre alternativas.

O projeto prioriza:

1. execução em CPU;
2. treinamento em CPU;
3. rastreabilidade;
4. reprodutibilidade;
5. integridade científica;
6. arquitetura simples;
7. experimentação controlada.

## Arquitetura aprovada

```
state + question + candidates
        ↓
Joint Packing
        ↓
LFM2.5-Encoder-230M
        ↓
Candidate Masks
        ↓
Attention Pooling
        ↓
256d / candidato
        ↓
Compact Set-Attention
        ↓
Regularized Evidential Head
        ↓
Dirichlet
        ↓
Conformal Selective Decision
        ↓
DECIDE / ABSTAIN
```

## Estado científico
Nenhum componente desta arquitetura deve ser tratado como superior antes de experimentação.

Os seguintes componentes são atualmente hipóteses:

```
Attention Pooling
Compact Set-Attention
Circle Loss
Regularized Evidential Head
Conformal Selective Decision
```

## Regra para resultados
Somente resultados provenientes de execução real podem ser apresentados como resultados do Kyn.

Estados permitidos:

```
NOT_RUN
RUNNING
MEASURED
VERIFIED
FAILED
INCONCLUSIVE
UNSPECIFIED
```

## Proibido
Nunca utilizar:

```
dados fabricados
resultados fabricados
telemetria simulada
métricas inventadas
benchmarks inventados
mocks como evidência científica
```

## Estado da implementação

M0 (estrutura do repositório) está em andamento. A base inicial de pacotes e dependências está sendo configurada; o encoder real, o processamento de candidatos, o treinamento e as decisões ainda estão `NOT_IMPLEMENTED`. Nenhum experimento foi executado (`NOT_RUN`) e nenhuma métrica foi medida (`NOT_MEASURED`).

O lockfile de dependências ainda não foi gerado: os índices de pacotes não estão acessíveis neste ambiente.
