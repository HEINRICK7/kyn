# Kyn Metrics Specification
As fórmulas das métricas devem ser únicas no projeto.

## Decision

```
Accuracy
Macro F1
Balanced Accuracy
```

## Probabilistic

```
Negative Log-Likelihood
Brier Score
Expected Calibration Error
```

## Selective

```
Coverage
Abstention Rate
Selective Risk
Risk-Coverage Curve
```

## CPU

```
latency p50
latency p95
latency p99
throughput
peak RAM
training duration
startup time
```

## Regra
Uma métrica só pode ser reportada se:

```
inputs reais existirem
cálculo tiver sido executado
resultado tiver sido persistido
```
Sem isso:

```
NOT_MEASURED
```
