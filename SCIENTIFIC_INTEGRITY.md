# Kyn Scientific Integrity

## 1. Regra máxima
O agente nunca deve inventar progresso para agradar o usuário.

## 2. Evidência
Todo resultado deve ser rastreável até:

```
code
commit
dataset
configuration
execution
raw artifact
metric
hardware
```

## 3. Proibições
É proibido fabricar:

```
resultados
metrics
logs
benchmarks
telemetry
checkpoints
dataset statistics
accuracy
loss
latency
RAM
CPU usage
plots
reports
```

## 4. Mock policy
Mocks não podem participar de evidência científica.

Podem existir somente para teste unitário quando explicitamente identificados.

## 5. Scientific run
Uma execução científica deve utilizar:

```
REAL MODEL
REAL DATA
REAL CODE
REAL EXECUTION
REAL METRICS
```

## 6. Sem execução
Se algo não foi executado:

```
NOT_RUN
```

## 7. Sem medição
Se algo não foi medido:

```
NOT_MEASURED
```

## 8. Sem verificação
Se algo não foi verificado:

```
NOT_VERIFIED
```

## 9. Falha
Falha deve ser registrada:

```
FAILED
```
Não converter falha em sucesso narrativo.

## 10. Inconclusivo
Quando resultados não sustentarem conclusão:

```
INCONCLUSIVE
```

## 11. Exemplos
Todo número puramente didático deve conter:

```
ILLUSTRATIVE_EXAMPLE
```

## 12. Resultados externos
Dados de papers devem incluir:

```
EXTERNAL_RESULT
```
Nunca tratar resultado externo como resultado do Kyn.

## 13. Dashboards
Dashboards só mostram dados reais.

Sem dados:

```
NO_DATA
```

## 14. Ciência negativa
Resultado negativo deve ser preservado.

Não apagar experimento porque ficou pior.
