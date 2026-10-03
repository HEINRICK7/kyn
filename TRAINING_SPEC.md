# Kyn Training Specification

## 1. Objetivo
Treinar o Kyn integralmente em CPU.

GPU não deve ser requisito.

## 2. T0 — Head Only
Congelar:

```
LFM2.5-Encoder-230M
```
Treinar:

```
Attention Pooling
Projection
Set-Attention
Evidential Head
```

## 3. T1 — Ablations
Comparar:

```
Mean Pooling
vs
Attention Pooling
```

```
MLP
vs
Set-Attention
```

```
Cross-Entropy
vs
Circle Loss
```

```
Softmax
vs
Evidential
```

## 4. T2 — Partial Unfreeze
Só pode acontecer se experimentação indicar necessidade.

O agente não deve decidir sozinho quantas camadas descongelar.

Deve primeiro inspecionar a arquitetura real do modelo.

Depois criar protocolo.

## 5. T3 — PEFT
LoRA ou adapters só podem entrar depois de:

```
1. verificar compatibilidade real;
2. identificar módulos reais;
3. criar protocolo;
4. registrar baseline.
```

## 6. T4 — Full Fine-Tuning
Não implementar inicialmente.

## 7. Optimizer inicial
Utilizar:

```
AdamW
```

## 8. Loss
Baseline:

```
Listwise Cross-Entropy
```
Candidata:

```
Listwise Circle Loss
```

## 9. Circle Loss
Circle Loss não pode substituir Cross-Entropy sem comparação controlada.

## 10. Precision
Primeiro baseline:

```
FP32
```
BF16 somente depois de:

```
hardware detection
benchmark
validation
```

## 11. Seeds
Todo experimento deve registrar seed.

Quando houver múltiplas seeds, reportar todas.

Nunca reportar somente a melhor seed.

## 12. Early stopping
Critério precisa ser definido antes da execução.

Não alterar patience depois de observar o resultado para produzir melhor número.

## 13. Checkpoints
Checkpoint só existe se arquivo real for gravado.

Nunca registrar:

```
checkpoint_saved = true
```
sem verificar existência física.
