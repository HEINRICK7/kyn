# Kyn Agent Rules
Este é o primeiro documento que qualquer agente deve ler.

## 1. Ordem de autoridade

```
1. SCIENTIFIC_INTEGRITY.md
2. AGENT_RULES.md
3. ARCHITECTURE.md
4. DATA_SPEC.md
5. TRAINING_SPEC.md
6. CPU_SPEC.md
7. EXPERIMENT_PROTOCOL.md
8. TESTING_STRATEGY.md
9. METRICS_SPEC.md
10. ROADMAP.md
11. DECISIONS.md
```

## 2. Seguir literalmente
O agente não possui autorização para alterar decisões registradas.

## 3. Regra operacional do Kyn
Se a especificação não estiver definida:

```
SPEC não definida = não inventar
```

Se o experimento não foi executado:

```
experimento não executado = NOT_RUN
```

Se o dado não foi medido:

```
dado não medido = NOT_MEASURED
```

Se a execução falhou:

```
falhou = FAILED
```

Nenhuma lacuna pode ser preenchida com plausibilidade, estimativa ou narrativa de sucesso.

## 4. Lacunas
Quando encontrar algo não especificado:

```
UNSPECIFIED
```
Não assumir.

## 5. Pesquisa externa
É permitido consultar documentação oficial, papers e código upstream para descobrir fatos técnicos.

Essas informações devem ser tratadas como:

```
EXTERNAL_EVIDENCE
```
e não como resultado do Kyn.

## 6. Código
Não criar código para fingir funcionalidade.

## 7. Placeholder
Placeholder deve ser explicitamente marcado:

```
PLACEHOLDER
NOT_IMPLEMENTED
```
Nunca fazê-lo parecer operacional.

## 8. Dados
Nunca inventar dataset.

## 9. Métricas
Nunca inventar resultado.

## 10. Dashboard
Nunca inventar telemetria.

## 11. Testes
Nunca usar mocks como evidência científica.

## 12. Erros
Se uma execução falhar, preservar:

```
command
stdout
stderr
exit code
```

## 13. Resultados
Todo resultado deve possuir caminho de artefato.

## 14. Relatórios
Relatórios devem distinguir:

```
FACT
EXTERNAL_EVIDENCE
HYPOTHESIS
MEASURED_RESULT
INTERPRETATION
LIMITATION
```

## 15. Proibição máxima
Nunca produzir uma narrativa de sucesso diferente do que os artefatos demonstram.

## 16. Regra final
Se houver escolha entre:

```
parecer que funciona
```
e:

```
mostrar que ainda não funciona
```
sempre escolher a segunda quando essa for a verdade.
