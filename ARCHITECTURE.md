# Kyn Architecture Specification

## 1. Princípio
A arquitetura deve ser implementada exatamente como especificada neste documento.

Mudanças arquiteturais exigem registro explícito em `DECISIONS.md`.

## 2. Backbone
Utilizar:

```
LFM2.5-Encoder-230M
```
O agente deve carregar o modelo real.

É proibido criar:

```
FakeLFM
MockEncoder
DummyEncoder
RandomEncoder
```
para demonstrar funcionamento científico.

## 3. Entrada
Formato conceitual:

```
state
question
candidates[]
```
Todos os candidatos participam do mesmo problema de decisão.

## 4. Joint Packing
Estado, pergunta e candidatos devem ser empacotados para uma execução conjunta.

O agente não pode substituir silenciosamente por processamento pointwise.

## 5. Candidate Masks
Cada token pertencente a candidato deve poder ser identificado.

Exemplo conceitual:

```
state/question = -1

candidate 0 = 0
candidate 1 = 1
candidate 2 = 2
```

## 6. Attention Pooling
Estratégia principal:

```
Attention Pooling
```
Cada candidato será reduzido a uma representação compacta.

Dimensão aprovada:

```
256
```

## 7. Controle
Implementar também:

```
Masked Mean Pooling
```
somente para comparação experimental.

## 8. Set-Attention
Configuração inicial:

```
dimension = 256
blocks = 1
heads = 4
```
Não aumentar número de blocos ou heads sem protocolo experimental.

## 9. Scorer
Scorer compartilhado:

```
256 → 64 → 1
```
O mesmo scorer deve ser utilizado para todos os candidatos.

## 10. Evidential Head
A saída principal deve permitir evidência não-negativa.

Utilizar ativação apropriada para garantir:

```
evidence >= 0
```
A formulação matemática precisa estar explicitamente documentada no código e nos testes.

## 11. Dirichlet
Para cada candidato:

```
alpha_i = evidence_i + 1
```
Probabilidade:

```
P_i = alpha_i / sum(alpha)
```

## 12. Conformal Decision
A camada conformal não pode acessar TEST durante calibração.

Deve utilizar exclusivamente o split definido como:

```
CALIBRATION
```

## 13. Saída
Estados finais:

```
DECIDE
ABSTAIN
```

## 14. Proibição
O agente não pode introduzir:

```
RAG
LLM generativo
agentes
memória externa
vector database
reinforcement learning
novo encoder
nova Decision Head
```
sem alteração explícita desta SPEC.
