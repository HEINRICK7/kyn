# Kyn Testing Strategy

## 1. TDD
Fluxo obrigatório:

```
RED
GREEN
REFACTOR
```

## 2. Domain tests
Testar:

```
candidate identity
permutation behavior
probability calculations
Dirichlet calculations
abstention rules
```

## 3. Candidate permutation
Teste obrigatório:

Dada uma entrada:

```
A B C D
```
e uma permutação:

```
C A D B
```
os scores devem acompanhar semanticamente os candidatos.

## 4. Candidate count
Testar quantidade variável:

```
2
3
4
8
N permitido pela arquitetura
```

## 5. Candidate masks
Verificar que tokens corretos pertencem ao candidato correto.

## 6. Integration tests
Precisam utilizar o `LFM2.5-Encoder-230M` real.

## 7. Mock boundary
Mocks podem testar aplicação ou domínio isolado.

Mocks não podem validar:

```
model quality
latency
accuracy
training
calibration
scientific result
```

## 8. Scientific evidence
Teste passando significa:

```
software behavior verified
```
Não significa:

```
scientific hypothesis confirmed
```
