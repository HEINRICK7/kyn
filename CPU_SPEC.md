# Kyn CPU Specification

## 1. Regra
CPU é o hardware primário do projeto.

## 2. Ambiente
Registrar:

```
CPU model
architecture
physical cores
logical threads
RAM
OS
kernel
Python
PyTorch
Transformers
```

## 3. FP32
FP32 é o baseline obrigatório.

## 4. BF16
BF16 só pode ser ativado depois de detectar suporte de hardware.

Nunca assumir suporte.

## 5. torch.compile
Comparar:

```
eager
vs
torch.compile
```
O agente não deve afirmar aceleração sem benchmark.

## 6. ONNX Runtime
Será utilizado apenas depois do modelo funcional em PyTorch.

## 7. OpenVINO
Opcional para benchmark CPU.

## 8. INT8
Candidato principal para inferência otimizada.

Após quantização medir novamente:

```
accuracy
NLL
Brier
ECE
coverage
selective risk
latency
RAM
```

## 9. INT4
Experimental.

Não ativar por padrão.

## 10. Benchmark
Registrar:

```
warmup iterations
measured iterations
threads
batch size
sequence length
candidate count
precision
runtime
```

## 11. Métricas

```
latency p50
latency p95
latency p99
throughput
RAM peak
startup time
```

## 12. Proibição
Não utilizar:

```
random latency
estimated throughput
fake CPU usage
fake RAM values
```
Todos os números precisam vir de medição.
