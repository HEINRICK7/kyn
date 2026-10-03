# Kyn Folder Architecture Specification

## 1. Objetivo
Este documento define a arquitetura oficial de pastas do Kyn.

O agente deve seguir esta estrutura exatamente.

Não criar novas pastas principais sem registrar uma decisão em `DECISIONS.md`.

Não mover responsabilidades entre camadas por conveniência.

A arquitetura deve preservar:

``
documentation?
```

```
presentation
    ↓
application
    ↓
domain

infrastructure → implementa contratos necessários
```
O domínio não pode depender de frameworks, banco, runtime de ML, Hugging Face, PyTorch, ONNX ou OpenVINO.

---

# 2. Estrutura oficial

```
kyn/
├── README.md
├── AGENT_RULES.md
├── ARCHITECTURE.md
├── FOLDER_ARCHITECTURE.md
├── DATA_SPEC.md
├── TRAINING_SPEC.md
├── CPU_SPEC.md
├── SCIENTIFIC_INTEGRITY.md
├── EXPERIMENT_PROTOCOL.md
├── TESTING_STRATEGY.md
├── METRICS_SPEC.md
├── ROADMAP.md
├── DECISIONS.md
├── KNOWN_LIMITATIONS.md
│
├── pyproject.toml
├── uv.lock / requirements.lock
├── .gitignore
├── .env.example
│
├── src/
│   └── kyn/
│       ├── domain/
│       ├── application/
│       ├── infrastructure/
│       └── presentation/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── architecture/
│   └── scientific_invariants/
│
├── configs/
│   ├── model/
│   ├── training/
│   ├── experiments/
│   └── runtime/
│
├── data/
│   ├── raw/
│   ├── external/
│   ├── processed/
│   ├── splits/
│   └── manifests/
│
├── models/
│   ├── upstream/
│   ├── checkpoints/
│   ├── exported/
│   └── manifests/
│
├── scientific/
│   ├── protocols/
│   ├── experiments/
│   ├── reports/
│   ├── ablations/
│   └── templates/
│
├── artifacts/
│   ├── runs/
│   ├── metrics/
│   ├── logs/
│   ├── predictions/
│   ├── calibration/
│   ├── benchmarks/
│   └── manifests/
│
├── scripts/
│   ├── data/
│   ├── training/
│   ├── evaluation/
│   ├── benchmark/
│   └── export/
│
└── docs/
    ├── research/
    ├── references/
    └── diagrams/
```

---

# 3. src/kyn/domain
Local das regras puras do sistema.

```
src/kyn/domain/
├── decision/
│   ├── entities.py
│   ├── value_objects.py
│   ├── policies.py
│   └── errors.py
│
├── uncertainty/
│   ├── evidence.py
│   ├── dirichlet.py
│   └── errors.py
│
├── calibration/
│   ├── prediction_set.py
│   ├── selective_decision.py
│   └── errors.py
│
└── metrics/
    ├── decision_metrics.py
    ├── probabilistic_metrics.py
    └── selective_metrics.py
```

## Pode conter

- entidades;
- value objects;
- invariantes;
- regras matemáticas;
- contratos conceituais;
- erros de domínio;
- políticas puras.

## Não pode conter

```
torch
transformers
huggingface
onnxruntime
openvino
pandas
FastAPI
filesystem
HTTP
CLI
```
O domínio deve ser testável sem carregar modelo algum.

---

# 4. src/kyn/application
Casos de uso e orquestração.

```
src/kyn/application/
├── ports/
│   ├── encoder.py
│   ├── model_repository.py
│   ├── dataset_repository.py
│   ├── artifact_store.py
│   └── hardware_probe.py
│
├── training/
│   ├── train_head.py
│   ├── evaluate_epoch.py
│   └── select_checkpoint.py
│
├── inference/
│   ├── decide.py
│   └── batch_decide.py
│
├── calibration/
│   └── calibrate_conformal.py
│
├── evaluation/
│   ├── evaluate_model.py
│   └── compare_runs.py
│
└── benchmark/
    ├── benchmark_inference.py
    └── benchmark_training.py
```

## Pode conter

- casos de uso;
- fluxo de treinamento;
- fluxo de inferência;
- chamadas para contratos;
- DTOs de entrada e saída;
- coordenação de experimentos.

## Não pode conter

- lógica específica de Hugging Face;
- leitura direta de arquivos;
- implementação ONNX;
- implementação OpenVINO;
- regras científicas escondidas.

---

# 5. src/kyn/infrastructure
Tudo que depende de tecnologia concreta.

```
src/kyn/infrastructure/
├── lfm/
│   ├── tokenizer.py
│   ├── encoder.py
│   ├── joint_packer.py
│   └── candidate_masks.py
│
├── pytorch/
│   ├── attention_pooling.py
│   ├── mean_pooling.py
│   ├── set_attention.py
│   ├── evidential_head.py
│   ├── losses.py
│   └── model.py
│
├── datasets/
│   ├── jsonl_repository.py
│   ├── parquet_repository.py
│   └── split_loader.py
│
├── calibration/
│   └── conformal.py
│
├── runtime/
│   ├── pytorch_runtime.py
│   ├── onnx_runtime.py
│   └── openvino_runtime.py
│
├── hardware/
│   └── cpu_probe.py
│
├── persistence/
│   ├── artifact_store.py
│   ├── checkpoint_store.py
│   └── manifest_store.py
│
└── export/
    ├── onnx_exporter.py
    └── openvino_exporter.py
```

## Regra
Toda dependência externa deve ficar aqui.

---

# 6. src/kyn/presentation
Camada de entrada/saída.

Inicialmente somente CLI.

```
src/kyn/presentation/
└── cli/
    ├── train.py
    ├── evaluate.py
    ├── infer.py
    ├── calibrate.py
    └── benchmark.py
```
Nada de regra de negócio nesta camada.

Exemplo permitido:

```
parse arguments
↓
call use case
↓
print result
```
Exemplo proibido:

```
parse arguments
↓
calculate Dirichlet
↓
load dataset
↓
train model
```

---

# 7. tests/unit
Testes rápidos e isolados.

```
tests/unit/
├── domain/
├── application/
└── infrastructure/
```
Mocks podem existir aqui quando necessários para testar contratos.

Esses testes não são evidência científica.

---

# 8. tests/integration
Testes com componentes reais.

```
tests/integration/
├── lfm_real_forward/
├── joint_packing/
├── training/
├── checkpoint/
└── runtime/
```
Para integração com backbone:

```
REAL LFM2.5
REAL TOKENIZER
REAL FORWARD
```
Não substituir por mock e chamar de integração.

---

# 9. tests/architecture
Testes que protegem a arquitetura.

```
tests/architecture/
├── test_domain_dependencies.py
├── test_application_dependencies.py
└── test_layer_boundaries.py
```
Exemplos:

- domain não pode importar torch;
- domain não pode importar infrastructure;
- presentation não pode acessar storage diretamente;
- application depende de ports.

---

# 10. tests/scientific_invariants
Testes de invariantes importantes para o modelo.

```
tests/scientific_invariants/
├── test_candidate_permutation.py
├── test_candidate_identity.py
├── test_variable_candidate_count.py
├── test_probability_normalization.py
└── test_abstention_consistency.py
```
Esses testes verificam comportamento estrutural.

Também não substituem experimento científico.

---

# 11. configs
Toda configuração utilizada em experimentos deve ser persistida.

```
configs/
├── model/
│   └── lfm2_5_encoder_230m.yaml
│
├── training/
│   ├── head_only.yaml
│   ├── circle_loss.yaml
│   └── cross_entropy.yaml
│
├── experiments/
│   └── exp001.yaml
│
└── runtime/
    ├── cpu_fp32.yaml
    ├── cpu_bf16.yaml
    └── cpu_int8.yaml
```

## Regra
Não esconder hiperparâmetros importantes no código.

Configuração de experimento deve ser serializável e versionável.

---

# 12. data/raw
Dados originais obtidos da fonte.

```
data/raw/
```
Nunca alterar arquivos brutos.

---

# 13. data/external
Dados externos baixados de terceiros.

```
data/external/
```
Devem possuir proveniência.

---

# 14. data/processed
Dados transformados.

```
data/processed/
```
Toda transformação deve ser reproduzível por script.

---

# 15. data/splits
Splits congelados.

```
data/splits/
├── train.jsonl
├── validation.jsonl
├── calibration.jsonl
└── test.jsonl
```
Futuramente:

```
blind.jsonl
```

## Regra
Não alterar splits silenciosamente.

---

# 16. data/manifests
Metadados dos datasets.

```
data/manifests/
└── dataset_manifest.json
```
Exemplo de campos:

```
source
license
hash
rows
split_counts
created_at
transformation_version
```

---

# 17. models/upstream
Pesos originais externos.

```
models/upstream/
```
Não modificar diretamente.

Registrar origem e revisão.

---

# 18. models/checkpoints
Checkpoints treinados pelo Kyn.

```
models/checkpoints/
└── EXP001/
```
Um checkpoint só deve existir se realmente foi gerado.

---

# 19. models/exported
Modelos exportados.

```
models/exported/
├── onnx/
└── openvino/
```
Não criar arquivos placeholder com extensão de modelo.

---

# 20. models/manifests
Proveniência dos modelos.

```
models/manifests/
```
Registrar:

```
upstream model
revision
hash
training experiment
checkpoint
export format
```

---

# 21. scientific/protocols
Protocolos definidos antes da execução.

```
scientific/protocols/
├── EXP001_HEAD_ONLY.md
├── EXP002_POOLING.md
└── ...
```

---

# 22. scientific/experiments
Registro final de cada experimento.

```
scientific/experiments/
└── EXP001/
    ├── protocol.md
    ├── result.md
    └── limitations.md
```

---

# 23. scientific/reports
Relatórios consolidados.

```
scientific/reports/
```
Devem derivar de experimentos reais.

---

# 24. scientific/ablations
Comparações controladas.

```
scientific/ablations/
├── pooling/
├── loss/
├── decision_head/
└── uncertainty/
```

---

# 25. scientific/templates
Templates oficiais.

```
scientific/templates/
├── EXPERIMENT_TEMPLATE.md
├── RESULT_TEMPLATE.md
└── ABLATION_TEMPLATE.md
```

---

# 26. artifacts/runs
Uma pasta por execução real.

```
artifacts/runs/
└── EXP001_RUN001/
```
Estrutura:

```
EXP001_RUN001/
├── command.txt
├── config.yaml
├── environment.json
├── stdout.log
├── stderr.log
├── exit_code.txt
└── run_manifest.json
```

---

# 27. artifacts/metrics
Métricas realmente calculadas.

```
artifacts/metrics/
```
Não armazenar números inventados.

---

# 28. artifacts/logs
Logs brutos reais.

```
artifacts/logs/
```
Não gerar logs sintéticos para parecer que houve execução.

---

# 29. artifacts/predictions
Predições reais do modelo.

```
artifacts/predictions/
```
Cada arquivo deve apontar para:

```
run_id
checkpoint
dataset
split
```

---

# 30. artifacts/calibration
Artefatos de calibração conformal.

```
artifacts/calibration/
```
Nunca misturar com TEST.

---

# 31. artifacts/benchmarks
Resultados de benchmark CPU.

```
artifacts/benchmarks/
```
Cada benchmark deve registrar hardware.

---

# 32. artifacts/manifests
Manifestos globais de artefatos.

```
artifacts/manifests/
```
Usados para rastreabilidade.

---

# 33. scripts/data
Scripts reproduzíveis para dados.

```
scripts/data/
├── download_dataset.py
├── validate_dataset.py
├── create_splits.py
└── build_manifest.py
```

---

# 34. scripts/training
Entrypoints de treino.

```
scripts/training/
└── train_head_only.py
```
O script deve chamar application use cases.

Não concentrar toda a regra dentro do script.

---

# 35. scripts/evaluation

```
scripts/evaluation/
├── evaluate.py
└── compare_runs.py
```

---

# 36. scripts/benchmark

```
scripts/benchmark/
├── benchmark_inference.py
└── benchmark_training.py
```
Nunca usar números hardcoded.

---

# 37. scripts/export

```
scripts/export/
├── export_onnx.py
└── export_openvino.py
```

---

# 38. docs/research
Notas de pesquisa externas.

```
docs/research/
```
Devem distinguir:

```
EXTERNAL_EVIDENCE
HYPOTHESIS
```

---

# 39. docs/references
Referências bibliográficas, links e metadados.

```
docs/references/
```

---

# 40. docs/diagrams
Diagramas técnicos.

```
docs/diagrams/
```
Diagramas não podem apresentar componentes ainda inexistentes como implementados.

Usar marcação:

```
PLANNED
IMPLEMENTED
EXPERIMENTAL
```

---

# 41. Dependências entre camadas
Permitido:

```
presentation → application
application → domain
infrastructure → application ports
infrastructure → domain
```
Proibido:

```
domain → infrastructure
domain → application
domain → presentation

application → presentation
application → implementação concreta de infrastructure
```

---

# 42. Regra para modelos PyTorch
Classes que herdam de:

```
torch.nn.Module
```
pertencem a:

```
infrastructure/pytorch/
```
Não ao domínio.

A matemática conceitual pode pertencer ao domínio.

A implementação concreta PyTorch não.

---

# 43. Regra para Hugging Face
Tudo que importa:

```
transformers
AutoModel
AutoTokenizer
LFM2.5
```
fica em:

```
infrastructure/lfm/
```
ou infraestrutura específica correspondente.

---

# 44. Regra para ONNX e OpenVINO
Devem ser adapters de infraestrutura.

O restante do sistema não deve saber qual runtime está sendo usado.

---

# 45. Regra para experimentos
Código experimental temporário não pode ser espalhado dentro de `src/`.

Se for necessário investigar algo isolado:

```
scientific/experiments/
```
ou:

```
scripts/
```
Depois, apenas código aprovado entra no núcleo.

---

# 46. Regra contra pastas genéricas
Evitar:

```
utils/
helpers/
common/
misc/
stuff/
```
Essas pastas tendem a virar depósito de responsabilidades.

Uma função deve pertencer ao contexto que representa.

---

# 47. Regra contra god modules
Evitar arquivos como:

```
model.py com 2000 linhas
training.py com tudo
utils.py com dezenas de responsabilidades
```
Separar por comportamento real.

---

# 48. Regra de criação de novas pastas
Antes de criar uma nova pasta, responder:

```
Qual responsabilidade ela representa?

Essa responsabilidade já existe em outra camada?

É domínio, aplicação, infraestrutura ou apresentação?

É código de produção, experimento ou artefato?

Ela precisa realmente existir?
```
Sem resposta clara, não criar.

---

# 49. Regra de integridade científica
Nunca misturar:

```
source code
experimental results
raw data
generated artifacts
scientific reports
```
Cada tipo possui seu próprio local.

Isso evita que um arquivo manual seja confundido com saída real de execução.

---

# 50. Estrutura mínima da primeira milestone
No início, não é necessário criar todos os arquivos vazios.

Criar somente o necessário:

```
kyn/
├── specs...
├── src/kyn/
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── presentation/
├── tests/
├── configs/
├── scientific/
├── artifacts/
└── scripts/
```
Pastas internas devem surgir conforme existirem responsabilidades reais.

Não criar centenas de arquivos placeholder.

---

# 51. Regra final
A arquitetura de pastas existe para tornar explícito:

```
o que é regra
o que é framework
o que é experimento
o que é dado
o que é resultado real
o que é documentação
```
Se uma nova organização tornar essa distinção menos clara, ela não deve ser adotada.

O agente deve preservar esta estrutura e não reorganizar o projeto por preferência pessoal.
