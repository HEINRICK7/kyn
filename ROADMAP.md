# Kyn Roadmap

## M0 — Repository
Criar estrutura, specs, testes básicos e configuração.

Nenhum resultado científico esperado.

## M1 — Real LFM Forward
Executar `LFM2.5-Encoder-230M` real em CPU.

Critério:

```
real weights loaded
real tokenizer
real forward
real tensors
```

## M2 — Candidate Packing
Implementar:

```
joint packing
candidate masks
```

## M3 — Pooling
Implementar:

```
Mean Pooling
Attention Pooling
```

## M4 — Decision Head
Implementar:

```
MLP baseline
Compact Set-Attention
```

## M5 — Training
Executar treino real head-only.

## M6 — Loss Ablation
Comparar:

```
Cross-Entropy
Circle Loss
```

## M7 — Evidential
Comparar:

```
Softmax
Evidential
Regularized Evidential
```

## M8 — Conformal
Implementar:

```
DECIDE
ABSTAIN
```

## M9 — CPU Optimization
Comparar:

```
FP32
BF16
torch.compile
ONNX
OpenVINO
INT8
```

## Regra
Não pular milestone alegando que etapa posterior já funciona sem evidência da anterior.
