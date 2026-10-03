# Kyn Data Specification

## 1. Formato canônico

```
{
  "context": {
    "state": "...",
    "question": "..."
  },
  "candidates": [
    {
      "id": "candidate_a",
      "text": "..."
    }
  ],
  "label": {
    "correct_candidate_id": "candidate_a"
  }
}
```

## 2. Identidade
A resposta correta deve ser referenciada pelo `candidate_id`.

Não utilizar índice como identidade permanente.

## 3. Candidate ordering
A ordem dos candidatos pode mudar sem alterar o significado do exemplo.

## 4. Candidate shuffling
Durante treinamento é permitido embaralhar candidatos.

O label continua apontando para o mesmo `candidate_id`.

## 5. Splits
Separação obrigatória:

```
TRAIN
VALIDATION
CALIBRATION
TEST
```
Opcional posteriormente:

```
BLIND
```

## 6. Anti-leakage
É proibido:

```
TRAIN ← dados de TEST
TRAIN ← dados de CALIBRATION
VALIDATION ← informações de TEST
CALIBRATION ← informações de TEST
```

## 7. Hard negatives
Configuração inicial:

```
1 positive
2 hard negatives
1 easy negative
```
Hard negatives devem ser reais ou produzidos por método documentado.

Não inventar candidatos artificialmente e apresentá-los como dados naturais.

## 8. Synthetic data
Dados sintéticos são permitidos somente quando explicitamente marcados:

```
data_type = SYNTHETIC
```
Nunca misturar sintético com real sem rastreabilidade.

## 9. Proveniência
Cada dataset precisa registrar:

```
name
source
license
version
download date
hash
transformations
number of examples
split counts
```

## 10. Valores ausentes
Se uma informação não existir:

```
null
```
ou:

```
UNKNOWN
```
Nunca preencher com um valor plausível.
