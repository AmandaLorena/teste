# Casos de Teste — Zona Azul Digital

## 1. Abertura e validação

| Cenário | Entrada | Resultado esperado |
|---|---|---|
| Abrir bilhete válido | Placa `ABC1D23` | HTTP 201 e status `aberto` |
| Placa ausente | Sem placa | HTTP 422, `placa_invalida` |
| Placa minúscula | `abc1d23` | HTTP 422, `placa_invalida` |
| Entrada inválida | Data fora de ISO-8601 | HTTP 422, `entrada_invalida` |
| Placa já ocupada | Abrir outro bilhete para placa com bilhete aberto | HTTP 409, `bilhete_em_aberto` |
| Reabrir após encerramento | Placa sem bilhete aberto | HTTP 201 |

## 2. Cobrança

Parâmetros: tarifa de 500 centavos por hora, fração de 15 minutos, teto de 6000 centavos e tolerância de 10 minutos.

| Duração | Resultado esperado |
|---:|---:|
| 0 minutos | 0 centavos |
| 10 minutos | 0 centavos |
| 11 minutos | 125 centavos |
| 15 minutos | 125 centavos |
| 16 minutos | 250 centavos |
| 30 minutos | 250 centavos |
| 60 minutos | 500 centavos |
| 720 minutos | 6000 centavos |
| 1440 minutos | 6000 centavos |

## 3. Encerramento e cancelamento

| Cenário | Resultado esperado |
|---|---|
| Encerrar bilhete aberto | HTTP 200 com `saida`, `minutos` e `valor_centavos` |
| Encerrar ID inexistente | HTTP 404, `bilhete_nao_encontrado` |
| Encerrar bilhete já encerrado | HTTP 409, `bilhete_ja_encerrado` |
| Cancelar bilhete aberto | HTTP 200 com status `cancelado` |
| Cancelar ID inexistente | HTTP 404, `bilhete_nao_encontrado` |
| Cancelar bilhete encerrado | HTTP 409, `bilhete_nao_aberto` |
| Cancelar bilhete cancelado | HTTP 409, `bilhete_nao_aberto` |

## 4. Consultas e relatório

| Cenário | Resultado esperado |
|---|---|
| Listar ativos | Apenas bilhetes abertos, mais recentes primeiro |
| Nenhum bilhete ativo | HTTP 200 e array vazio |
| Histórico de placa conhecida | Todos os estados, mais recentes primeiro |
| Histórico de placa desconhecida | HTTP 200 e array vazio |
| Relatório com data válida | HTTP 200 com total, faturamento e média |
| Data inválida | HTTP 422, `data_invalida` |
| Média de 10,5 minutos | `tempo_medio_minutos: 11` |

## 5. Precedência e integridade

| Cenário | Resultado esperado |
|---|---|
| Placa inválida com bilhete existente para texto semelhante | HTTP 422, `placa_invalida` |
| Valor monetário | Sempre inteiro em centavos |
| Cobrança acima do teto | Nunca ultrapassa 6000 centavos |
| Cancelamento | Não gera cobrança, `saida` ou `valor_centavos` |

Os testes devem verificar tanto o status HTTP quanto o corpo JSON completo quando definido pelo contrato.
