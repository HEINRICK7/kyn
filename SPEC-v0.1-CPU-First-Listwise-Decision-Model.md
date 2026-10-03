# SPEC v0.1 — CPU-First Listwise Decision Model

## 1. Objetivo
Construir um modelo de decisão não-autoregressivo, CPU-first, baseado no `LFM2.5-Encoder-230M`, capaz de:

- receber um contexto;
- receber uma pergunta;
- receber um conjunto variável de candidatos;
- avaliar todos os candidatos conjuntamente;
- produzir scores listwise;
- produzir probabilidades;
- estimar incerteza;
- decidir ou se abster quando não houver evidência suficiente.
O projeto não será construído como chatbot, agente ou modelo generativo.

O objetivo central é:

> transformar um encoder bidirecional eficiente em CPU em um sistema especializado de decisão conjunta, probabilística e seletiva.

---

# 2. Princípios obrigatórios

## 2.1 CPU-first
Todo o núcleo deve ser:

- executável em CPU;
- treinável em CPU;
- testável em CPU;
- benchmarkável em CPU.
GPU pode futuramente ser usada como acelerador opcional, mas nunca será requisito arquitetural.

---

## 2.2 Não autoregressivo
O sistema não deve gerar tokens para tomar decisões.

Fluxo esperado:

```
input
 ↓
encoder
 ↓
representações
 ↓
decision head
 ↓
scores
 ↓
probabilidades
 ↓
decisão
```
Não:

```
input
 ↓
LLM
 ↓
gera texto
 ↓
parseia resposta
```

---

## 2.3 Evidência acima de opinião
Toda mudança arquitetural deverá ser tratada como hipótese.

Nenhuma técnica será considerada melhor sem experimento controlado.

Estados possíveis de uma hipótese:

```
PROPOSED
TESTING
ACCEPTED
REJECTED
INCONCLUSIVE
```

---

# 3. Backbone
Backbone selecionado:

```
LFM2.5-Encoder-230M
```
O backbone será inicialmente congelado.

Objetivos:

- reduzir custo de treinamento;
- permitir treino em CPU;
- isolar o efeito da Decision Head;
- facilitar experimentos reproduzíveis.

---

# 4. Contrato de entrada
O dataset não utilizará posição como identidade semântica da resposta correta.

Formato canônico:

```
{
  "context": {
    "state": "Cliente relata duas cobranças idênticas no cartão.",
    "question": "Qual setor deve tratar o caso?"
  },
  "candidates": [
    {
      "id": "opt_a",
      "text": "Suporte geral"
    },
    {
      "id": "opt_b",
      "text": "Cobrança"
    },
    {
      "id": "opt_c",
      "text": "Fraude"
    },
    {
      "id": "opt_d",
      "text": "Cancelamento"
    }
  ],
  "label": {
    "correct_candidate_id": "opt_b"
  }
}
```
Não utilizar como representação principal:

```
{
  "target": 1
}
```
porque índices dependem da posição dos candidatos.

---

# 5. Invariância de posição
A identidade de um candidato será determinada por seu `id`, não por sua posição.

Exemplo:

```
A B C D
```
e:

```
C A D B
```
devem representar semanticamente o mesmo problema.

A saída deverá apenas acompanhar a permutação dos candidatos.

Essa propriedade será testada explicitamente.

---

# 6. Joint Packing
`state`, `question` e todos os candidatos serão processados conjuntamente.

Representação conceitual:

```
[STATE]
...

[QUESTION]
...

[CANDIDATE]
...

[CANDIDATE]
...

[CANDIDATE]
...
```
O tokenizer/packer deverá também produzir uma máscara indicando quais tokens pertencem a cada candidato.

Exemplo:

```
state/question → -1
candidate A    → 0
candidate B    → 1
candidate C    → 2
candidate D    → 3
```

---

# 7. Candidate Masks
Depois do encoder, cada candidato deverá ter seus tokens identificáveis independentemente.

Fluxo:

```
joint sequence
      ↓
LFM2.5
      ↓
hidden states
      ↓
candidate masks
      ↓
tokens de cada candidato
```
Isso permite aplicar pooling individual sem executar novamente o encoder.

---

# 8. Option Compression
Estratégia principal:

```
Attention Pooling
```
Cada candidato deverá ser comprimido para um vetor de:

```
256 dimensões
```
Fluxo:

```
candidate token representations
        ↓
Attention Pooling
        ↓
projection
        ↓
256d
```

---

# 9. Baseline de compressão
Controle obrigatório:

```
Masked Mean Pooling
```
Comparação:

```
C0 = Mean Pooling
C1 = Attention Pooling
```
Attention Pooling só permanece se justificar a complexidade experimentalmente.

---

# 10. Decision Model
Modelo selecionado:

```
Joint / Listwise Decision Model
```
Todos os candidatos participam da decisão conjuntamente.

Não utilizar como modelo principal:

```
(state, option A) → forward
(state, option B) → forward
(state, option C) → forward
```
porque isso transforma o custo em aproximadamente `N forwards`.

O objetivo é:

```
state + question + N candidates
              ↓
          1 encoder
              ↓
       N representações
              ↓
       joint comparison
```

---

# 11. Decision Head
Decision Head selecionada:

```
Compact Set-Attention Head
```
Configuração inicial:

```
input dimension: 256
attention blocks: 1
attention heads: 4
scorer MLP: 256 → 64 → 1
```
Fluxo:

```
candidate A 256d ─┐
candidate B 256d ─┤
candidate C 256d ─┼─→ Set Self-Attention
candidate D 256d ─┘
                         ↓
                    LayerNorm
                         ↓
                    shared scorer
                         ↓
                1 score/candidate
```

---

# 12. Shared Scorer
Todos os candidatos deverão utilizar o mesmo scorer.

Não haverá uma cabeça diferente para cada posição.

Exemplo:

```
candidate representation
        ↓
Linear 256 → 64
        ↓
activation
        ↓
Linear 64 → 1
        ↓
score
```
Isso permite quantidade variável de candidatos.

---

# 13. Controle da Decision Head
Baseline:

```
candidate 256d
    ↓
shared MLP
    ↓
score
```
Experimento:

```
candidate set
    ↓
Set-Attention
    ↓
shared MLP
    ↓
score
```
Comparação:

```
H0 = MLP scorer
H1 = Compact Set-Attention
```
Se Set-Attention não apresentar benefício mensurável, deverá ser removida.

---

# 14. Modelo de incerteza
A saída principal não utilizará apenas Softmax como interpretação de confiança.

Estratégia selecionada:

```
Regularized Evidential Head
```
Cada candidato produz evidência não-negativa.

Exemplo conceitual:

```
candidate
   ↓
256d
   ↓
MLP
   ↓
Softplus
   ↓
evidence ≥ 0
```

---

# 15. Distribuição Dirichlet
Para cada candidato:

```
alpha_i = evidence_i + 1
```
Probabilidade estimada:

```
P_i = alpha_i / Σ alpha
```
A concentração total deverá ser utilizada como informação sobre quantidade de evidência disponível.

O sistema deverá diferenciar:

```
qual candidato parece melhor
```
de:

```
quanta evidência sustenta essa decisão
```

---

# 16. Controle de incerteza
Comparação obrigatória:

```
U0 = Softmax tradicional
U1 = Evidential
U2 = Regularized Evidential
```
Métricas de calibração deverão determinar qual estratégia permanece.

---

# 17. Abstention
O modelo poderá recusar uma decisão.

Estados finais:

```
DECIDE
ABSTAIN
```
Não será utilizado apenas um threshold arbitrário fixo como regra definitiva.

---

# 18. Conformal Selective Decision
Estratégia selecionada para abstention:

```
Conformal Prediction / Conformal Abstention
```
Fluxo:

```
Evidential Head
       ↓
probabilidades/evidência
       ↓
conformal calibration
       ↓
prediction set
```
Regra inicial:

```
prediction set com 1 candidato
→ DECIDE

prediction set com >1 candidato
→ ABSTAIN
```

---

# 19. Calibration Split
Conformal Prediction exigirá uma partição separada de calibração.

Partições:

```
TRAIN
VALIDATION
CALIBRATION
TEST
```
Uma futura avaliação blind poderá adicionar:

```
BLIND
```
Dados de `TEST` ou `BLIND` nunca poderão ser utilizados para calibrar thresholds.

---

# 20. Formação dos candidatos durante treino
Configuração inicial:

```
1 positive
2 hard negatives
1 easy negative
```
Exemplo:

```
positive:
Cobrança

hard negatives:
Fraude
Contestação de pagamento

easy negative:
Marketing
```

---

# 21. Hard negatives
Hard negatives deverão ser semanticamente plausíveis.

Não deverão ser criados apenas para aumentar artificialmente a dificuldade.

Precisam representar alternativas realisticamente confundíveis com a correta.

---

# 22. Candidate shuffling
A ordem dos candidatos deverá ser embaralhada durante o treinamento.

Objetivos:

- evitar position bias;
- impedir atalhos estatísticos;
- validar a invariância da Decision Head.
O `correct_candidate_id` permanece constante.

---

# 23. Loss principal
Candidata principal:

```
Listwise Circle Loss
```
Objetivo:

- separar positivos dos negativos;
- dar maior peso a negativos difíceis;
- explorar o contexto listwise.

---

# 24. Loss de controle
Baseline obrigatório:

```
Listwise Cross-Entropy
```
Comparação:

```
L0 = Cross-Entropy
L1 = Circle Loss
```
Circle Loss não será mantida apenas por ser mais recente.

Ela deverá produzir benefício mensurável.

---

# 25. Regime de treinamento CPU-first
Treinamento será dividido em estágios.

## T0 — Head Only

```
LFM2.5 → FROZEN

Attention Pooling → TRAINABLE
Projection        → TRAINABLE
Set-Attention     → TRAINABLE
Evidential Head   → TRAINABLE
```
Primeiro estágio obrigatório.

---

# 26. T1 — Head Optimization
Depois do funcionamento básico:

- comparar CE vs Circle Loss;
- comparar Mean vs Attention Pooling;
- comparar MLP vs Set-Attention;
- comparar Softmax vs Evidential.
Backbone continua congelado.

---

# 27. T2 — Partial Unfreezing
Só será executado se T0/T1 apresentarem evidência de limitação na representação.

Estratégia:

```
início do encoder → frozen
meio              → frozen
última parte      → trainable
decision head     → trainable
```
A porcentagem exata será definida após inspeção real da arquitetura do LFM2.5.

Nenhum número de layers deverá ser inventado antecipadamente.

---

# 28. T3 — Parameter Efficient Fine-Tuning
Antes de full fine-tuning poderão ser avaliados:

```
LoRA
Adapters
PEFT equivalente compatível
```
Somente depois de verificar os módulos reais disponíveis na arquitetura LFM2.5.

---

# 29. T4 — Full Fine-Tuning
Full fine-tuning dos 230M parâmetros:

```
NÃO faz parte da primeira implementação.
```
Só será realizado caso experimentos anteriores indiquem necessidade real.

---

# 30. Precisão numérica
Baseline:

```
FP32
```
Depois:

```
BF16
```
somente se a CPU possuir suporte eficiente.

Não assumir que BF16 é automaticamente mais rápido.

Hardware deverá ser detectado e benchmarkado.

---

# 31. torch.compile
`torch.compile` será avaliado experimentalmente.

Comparação:

```
PyTorch eager
vs
torch.compile
```
Só será ativado permanentemente se produzir benefício mensurável na CPU alvo.

---

# 32. Optimizer
Inicialmente:

```
AdamW
```
Head-only pode utilizar learning rate significativamente maior que o encoder.

Valores definitivos serão definidos pelo primeiro protocolo de treinamento.

---

# 33. Optimizers quantizados
Futuramente:

```
AdamW8bit
```
poderá ser comparado ao AdamW normal principalmente durante partial fine-tuning.

Não utilizar automaticamente.

---

# 34. Quantização
Quantização será aplicada inicialmente à inferência, não ao treinamento.

Ordem:

```
FP32 baseline
↓
BF16, quando apropriado
↓
INT8
↓
INT4 experimental
```
INT8 será o principal alvo de produção.

---

# 35. Runtime CPU
Runtimes a serem comparados futuramente:

```
PyTorch
ONNX Runtime
OpenVINO
```
Objetivo:

identificar o runtime com melhor relação entre:

- latência;
- throughput;
- RAM;
- fidelidade numérica;
- simplicidade operacional.

---

# 36. INT8
INT8 deverá ser tratado como candidato principal para inferência otimizada.

Após quantização deverão ser reavaliados:

- accuracy;
- ranking;
- calibração;
- abstention;
- selective risk.
Não basta verificar apenas velocidade.

---

# 37. INT4
INT4 será experimental.

Somente será mantido se preservar qualidade suficiente.

Nenhuma decisão arquitetural dependerá de INT4.

---

# 38. Arquitetura completa v0.1

```
               INPUT
                 │
        ┌────────┴────────┐
        │                 │
      state            question
        │                 │
        └────────┬────────┘
                 │
             candidates[]
                 │
                 ▼
            Joint Packing
                 │
                 ▼
       LFM2.5-Encoder-230M
             FROZEN
                 │
                 ▼
           Hidden States
                 │
                 ▼
         Candidate Masks
                 │
                 ▼
        Attention Pooling
                 │
                 ▼
        256d / Candidate
                 │
                 ▼
     Compact Set-Attention
          1 block / 4 heads
                 │
                 ▼
      Regularized Evidential
             Scorer
                 │
                 ▼
       Evidence / Candidate
                 │
                 ▼
             Dirichlet
                 │
                 ▼
      Probability + Uncertainty
                 │
                 ▼
      Conformal Calibration
                 │
                 ▼
          ┌──────┴──────┐
          ▼             ▼
       DECIDE         ABSTAIN
```

---

# 39. Separação arquitetural do projeto
Estrutura inicial:

```
src/
├── domain/
│   ├── decision/
│   ├── uncertainty/
│   ├── calibration/
│   └── errors/
│
├── application/
│   ├── training/
│   ├── evaluation/
│   ├── inference/
│   └── benchmark/
│
├── infrastructure/
│   ├── lfm/
│   ├── datasets/
│   ├── pytorch/
│   ├── onnx/
│   └── openvino/
│
└── presentation/
    └── cli/
```
Frameworks devem permanecer nas bordas.

---

# 40. Scientific Records
O projeto deverá possuir:

```
scientific/
```
Todo experimento relevante terá um documento próprio.

Exemplo:

```
scientific/
├── EXP001_HEAD_ONLY.md
├── EXP002_POOLING_ABLATION.md
├── EXP003_SET_ATTENTION.md
├── EXP004_LOSS_ABLATION.md
├── EXP005_EVIDENTIAL.md
├── EXP006_CONFORMAL.md
└── EXP007_CPU_RUNTIME.md
```

---

# 41. Estrutura obrigatória de experimento
Cada experimento deverá registrar:

```
ID
data
commit hash
hipótese
baseline
alteração
dataset
splits
seed
hardware
CPU
RAM
software
configuração
resultados
métricas
artefatos
conclusão
limitações
status
```

---

# 42. Métricas de decisão
Obrigatórias inicialmente:

```
Accuracy
Macro F1
Balanced Accuracy
```
Dependendo do dataset poderão ser adicionadas métricas específicas.

---

# 43. Métricas probabilísticas
Obrigatórias:

```
Negative Log-Likelihood
Brier Score
Expected Calibration Error
```

---

# 44. Métricas seletivas
Obrigatórias:

```
Coverage
Abstention Rate
Selective Risk
Risk-Coverage Curve
```

---

# 45. Métricas CPU
Todo benchmark deverá registrar:

```
latency p50
latency p95
latency p99
throughput
peak RAM
model size
startup time
training time
```

---

# 46. Ambiente de benchmark
Registrar obrigatoriamente:

```
CPU model
physical cores
logical threads
RAM
OS
Python
PyTorch
Transformers
runtime
precision
thread configuration
commit hash
seed
```
Resultados sem hardware identificado não serão considerados comparáveis.

---

# 47. Testes obrigatórios

## Domain tests

- decisão correta;
- invariantes;
- cálculo de probabilidades;
- cálculo de incerteza;
- abstention.

## Architecture tests

- quantidade variável de candidatos;
- candidato correto não depende da posição;
- shapes;
- candidate masks;
- pooling.

## Scientific invariants
Testar obrigatoriamente:

```
permutation equivariance
```
Ao permutar candidatos:

```
scores devem ser permutados da mesma forma.
```

---

# 48. Anti-leakage
É proibido:

- treinar com TEST;
- calibrar usando TEST;
- escolher hiperparâmetros usando BLIND;
- reutilizar BLIND depois de observar resultados e ainda chamá-lo de blind;
- gerar hard negatives a partir de informações do gold de avaliação.

---

# 49. Critério de simplicidade
Uma técnica só permanece se:

- melhorar qualidade;
- reduzir custo;
- melhorar calibração;
- melhorar seletividade;
- ou resolver limitação concreta.
Se duas soluções tiverem qualidade equivalente, escolher a mais simples e barata.

---

# 50. Fora do escopo da v0.1
Não implementar agora:

- RAG;
- vector database;
- agentes;
- LangChain;
- banco de dados;
- microservices;
- Kafka;
- Kubernetes;
- memória externa;
- geração textual;
- reinforcement learning;
- distributed training;
- GPU-specific architecture;
- full fine-tuning prematuro.

---

# 51. Stack inicial
Mínimo necessário:

```
Python
PyTorch
Transformers
NumPy
SciPy
scikit-learn
pytest
safetensors
```
Posteriormente:

```
ONNX Runtime
OpenVINO
TorchAO
```
somente quando entrarem nos experimentos correspondentes.

---

# 52. Regra para dependências
Não adicionar dependência sem responder:

1. qual problema resolve?
2. por que não podemos resolver de forma simples?
3. afeta domínio ou somente infraestrutura?
4. aumenta custo de execução?
5. dificulta reprodução?
6. existe benefício mensurável?

---

# 53. Filosofia de implementação
Fluxo obrigatório:

```
comportamento
↓
spec
↓
teste
↓
implementação mínima
↓
experimento
↓
evidência
↓
refactor
```

---

# 54. Primeira milestone
A primeira milestone não será “modelo inteligente”.

Será:

> Executar um exemplo real completo em CPU com LFM2.5 congelado, quantidade variável de candidatos, candidate masks corretas, Attention Pooling e Decision Head treinável.
Critérios:

```
CPU only
sem mocks do modelo
forward real
backward real na head
loss real
checkpoint real
resultado reproduzível
```

---

# 55. Segunda milestone
Treinar a Decision Head em dataset real.

Comparar:

```
Mean Pooling
vs
Attention Pooling
```
e:

```
MLP scorer
vs
Set-Attention scorer
```

---

# 56. Terceira milestone
Comparar:

```
Cross-Entropy
vs
Circle Loss
```

---

# 57. Quarta milestone
Adicionar:

```
Regularized Evidential Head
```
e comparar com Softmax.

---

# 58. Quinta milestone
Adicionar:

```
Conformal Selective Decision
```
e medir:

```
coverage × risk
```

---

# 59. Sexta milestone
CPU optimization:

```
FP32
vs
BF16
vs
torch.compile
vs
ONNX
vs
OpenVINO
vs
INT8
```
Cada comparação deverá preservar o mesmo modelo e o mesmo dataset.

---

# 60. Definição de sucesso
O projeto será considerado tecnicamente bem-sucedido quando existir evidência reproduzível de que ele:

1. aprende uma tarefa de decisão real;
2. roda integralmente em CPU;
3. aceita número variável de candidatos;
4. não depende da posição dos candidatos;
5. produz scores listwise;
6. possui probabilidades avaliáveis;
7. representa incerteza melhor que Softmax simples ou rejeita essa hipótese;
8. consegue se abster de forma calibrada;
9. possui benchmark CPU reproduzível;
10. mantém arquitetura simples o suficiente para entender onde cada ganho surgiu.

---

# 61. Regra científica final
Nenhum componente desta SPEC é considerado definitivamente superior antes de ser testado.

Itens como:

```
Attention Pooling
Set-Attention
Circle Loss
Evidential Head
Conformal Abstention
BF16
torch.compile
INT8
```

são hipóteses ou candidatos tecnológicos.

O projeto deve poder concluir:

```
FAIL
```
e remover uma técnica sem comprometer sua arquitetura fundamental.

O objetivo não é provar que nossas ideias estão certas.

O objetivo é descobrir quais delas realmente funcionam.

---

# 62. Regra de Integridade Científica do Kyn

## 62.1. Princípio

O Kyn deve preservar valor científico acima de aparência de progresso.

O agente responsável por criar, alterar, testar ou documentar o projeto não pode fabricar evidência.

Se algo não foi executado, medido ou verificado, deve ser declarado explicitamente como tal.

---

## 62.2. Proibições absolutas

É proibido:

- inventar métricas;
- inventar resultados;
- inventar benchmarks;
- inventar tempos de execução;
- inventar consumo de CPU ou RAM;
- inventar accuracy, loss, F1, ECE, Brier ou qualquer outra métrica;
- inventar logs de treinamento;
- inventar respostas do modelo;
- inventar checkpoints;
- inventar datasets;
- inventar APIs ou integrações;
- inventar arquivos como se tivessem sido realmente gerados;
- inventar execução de experimentos;
- simular sucesso para apresentar o projeto como funcional;
- preencher campos ausentes com números plausíveis;
- copiar números de artigos e apresentá-los como resultados do Kyn;
- alterar critérios depois de observar resultados para transformar FAIL em PASS;
- esconder regressões;
- selecionar apenas seeds favoráveis;
- descartar resultados negativos sem justificativa documentada.

---

## 62.3. Mocks

Mocks, fakes e stubs não podem ser usados como evidência científica.

Exemplo proibido:

```text
FakeLFMEncoder
        ↓
scores inventados
        ↓
dashboard
        ↓
"modelo funcionando"
```

Isso não demonstra funcionamento do Kyn.

Quando o objetivo do teste for validar regra de domínio isolada, um test double pode ser usado somente se:

1. estiver explicitamente identificado como test double;
2. o teste não for utilizado como evidência de desempenho do modelo;
3. nenhuma métrica científica depender dele;
4. existir posteriormente um teste de integração com o componente real.

Para experimentos científicos:

```text
REAL MODEL
REAL DATA
REAL EXECUTION
REAL METRICS
```

---

## 62.4. Estados permitidos para resultados

Todo resultado deve pertencer a uma destas categorias:

```text
MEASURED
VERIFIED
NOT_MEASURED
NOT_VERIFIED
FAILED
INCONCLUSIVE
```

Nunca substituir ausência de informação por uma estimativa apresentada como fato.

Exemplo correto:

```json
{
  "training_time_seconds": null,
  "status": "NOT_MEASURED"
}
```

Exemplo proibido:

```json
{
  "training_time_seconds": 182
}
```

quando `182` não foi realmente medido.

---

## 62.5. Exemplos ilustrativos

Exemplos didáticos são permitidos, mas devem ser explicitamente identificados.

Permitido:

```text
EXEMPLO ILUSTRATIVO — NÃO É RESULTADO DO KYN

A: 0.10
B: 0.75
C: 0.15
```

Proibido apresentar o mesmo conteúdo como:

```text
Resultado do Kyn:

A: 0.10
B: 0.75
C: 0.15
```

sem execução real correspondente.

---

## 62.6. Resultados derivados de artigos

Números vindos de papers, documentação ou terceiros devem ser atribuídos à fonte.

Exemplo:

```text
O paper X reporta 25% de melhoria no benchmark Y.
```

Nunca:

```text
Kyn ficou 25% mais rápido.
```

até o Kyn reproduzir o experimento.

---

## 62.7. Proveniência

Todo resultado científico do Kyn deverá ser rastreável.

Registrar no mínimo:

```text
experiment_id
timestamp
git_commit
dataset
dataset_hash
split
seed
model
model_revision
model_hash quando aplicável
configuration
CPU
RAM
OS
Python version
PyTorch version
dependencies
command executed
raw output
metrics
checkpoint
status
```

---

## 62.8. Artefatos brutos

Resultados consolidados não são suficientes.

Sempre que possível manter:

```text
raw/
metrics/
logs/
checkpoints/
reports/
configs/
```

O relatório deve ser reconstruível a partir dos artefatos brutos.

---

## 62.9. Dataset real

Datasets utilizados em experimentos devem existir fisicamente e possuir proveniência.

Registrar:

```text
origem
licença
versão
hash
quantidade de exemplos
splits
transformações aplicadas
```

Dados sintéticos podem ser utilizados apenas quando o protocolo declarar explicitamente:

```text
DATA_TYPE = SYNTHETIC
```

e nunca devem ser chamados de dados reais.

---

## 62.10. Sem leakage

É proibido utilizar informação de:

```text
TEST
CALIBRATION
BLIND
```

durante treinamento quando isso violar o protocolo.

Especialmente:

```text
TRAIN → treinamento
VALIDATION → seleção
CALIBRATION → calibração
TEST → avaliação final
BLIND → avaliação congelada
```

---

## 62.11. Congelamento de protocolos

Antes de um experimento importante, registrar:

```text
hipótese
dataset
métricas
baseline
critério de PASS
critério de FAIL
```

Depois da execução, os critérios não podem ser alterados retroativamente para favorecer o resultado.

---

## 62.12. Resultados negativos

FAIL possui o mesmo valor científico que PASS.

Exemplo válido:

```text
EXP004 — Circle Loss

Resultado:
- Accuracy inferior à Cross-Entropy.
- Calibração pior.
- Tempo de treinamento maior.

STATUS: REJECTED
```

A implementação pode então ser removida.

Não tentar justificar uma técnica apenas porque ela fazia parte da proposta inicial.

---

## 62.13. Comparações

Toda comparação deverá controlar variáveis relevantes.

Por exemplo:

```text
mesmo dataset
mesmo split
mesma seed
mesma CPU
mesmo número de exemplos
mesmo contexto máximo
mesmo protocolo
```

Quando isso não for possível, a limitação deverá aparecer no relatório.

---

## 62.14. Dashboard

O dashboard do Kyn só poderá mostrar fatos provenientes de artefatos reais.

Se não houver execução:

```text
NO DATA
```

Se não houver métrica:

```text
NOT MEASURED
```

Se treinamento estiver parado:

```text
IDLE
```

Nunca mostrar animações ou progresso fictício como se fossem execução real.

---

## 62.15. Telemetria

CPU, memória, latência e throughput devem vir de instrumentação real.

Nunca gerar valores aleatórios para tornar o dashboard visualmente ativo.

Permitido:

```text
cpu_percent = valor lido do sistema
```

Proibido:

```python
cpu_percent = random.uniform(40, 80)
```

em qualquer fluxo apresentado como telemetria real.

---

## 62.16. Testes

Testes unitários podem testar comportamento isolado.

Mas:

```text
unit test PASS
```

não significa:

```text
modelo cientificamente validado
```

Distinguir sempre:

```text
software correctness
vs
scientific evidence
```

---

## 62.17. Hipóteses

Arquiteturas escolhidas atualmente são hipóteses.

Exemplos:

```text
Attention Pooling
Compact Set-Attention
Circle Loss
Regularized Evidential Head
Conformal Selective Decision
```

Nenhuma delas deve aparecer em documentação como comprovadamente superior antes dos experimentos.

---

## 62.18. Linguagem obrigatória

Quando não houver evidência suficiente, utilizar termos como:

```text
hipótese
candidato
proposto
não testado
não medido
não verificado
resultado inconclusivo
```

Evitar:

```text
comprovado
superior
mais eficiente
mais inteligente
melhor
```

sem dados que sustentem a afirmação.

---

## 62.19. Regra para o agente

Antes de afirmar qualquer resultado, o agente deve conseguir responder:

```text
Onde está o artefato?

Qual comando produziu isso?

Qual dataset foi usado?

Qual configuração?

Qual commit?

Qual hardware?

Qual métrica bruta sustenta a conclusão?
```

Se não conseguir responder, não pode afirmar o resultado.

---

## 62.20. Regra máxima do Kyn

> O agente nunca deve inventar progresso para agradar o usuário.

Um resultado real ruim vale mais que um resultado excelente fabricado.

Quando o Kyn falhar, registrar FAIL.

Quando não soubermos, registrar UNKNOWN.

Quando não tivermos executado, registrar NOT RUN.

Quando não houver evidência, não concluir.

A credibilidade científica do projeto tem prioridade sobre qualquer demonstração visual, expectativa ou narrativa de sucesso.
