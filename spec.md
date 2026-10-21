# Especificação Funcional — Zona Azul Digital

## 1. Objetivo e escopo

Implementar uma API REST para abertura, encerramento, cancelamento e consulta de bilhetes de estacionamento rotativo, incluindo histórico por placa e relatório diário.

A aplicação não deve incluir interface gráfica ou funcionalidades de back-office.

## 2. Configuração

- Base URL: `http://localhost:{PORTA_SERVICO}`.
- A porta deve ser obtida de `variante/params.json`.
- Utilizar o fuso horário `-03:00`.
- Utilizar `TARIFA_HORA_CENTAVOS`, `FRACAO_MINUTOS`, `TETO_DIARIO_CENTAVOS` e `TOLERANCIA_MINUTOS` conforme `variante/params.json`.
- Não exigir variáveis de ambiente adicionais para executar o serviço.

## 3. Modelo de dados

Cada bilhete deve possuir:

- `id`: identificador único.
- `placa`: placa do veículo.
- `entrada`: data e hora de abertura.
- `status`: estado do bilhete.

Estados possíveis:

- `aberto`: ainda não encerrado nem cancelado.
- `encerrado`: estacionamento finalizado e cobrança calculada.
- `cancelado`: bilhete cancelado sem cobrança.

Bilhetes encerrados devem possuir `saida`, `minutos` e `valor_centavos`. Bilhetes cancelados não devem possuir `saida` nem `valor_centavos`.

## 4. UC1 — Abrir bilhete

**Endpoint:** `POST /bilhetes`

### Entrada

JSON contendo:

- `placa`: obrigatória, com exatamente sete caracteres alfanuméricos maiúsculos.
- `entrada`: opcional, em ISO-8601 com fuso horário. Quando ausente, utilizar o instante atual.

### Sucesso

Retornar HTTP 201 com:

`{"id": 1, "placa": "ABC1D23", "entrada": "2026-10-12T08:30:00-03:00", "status": "aberto"}`

O identificador deve ser único. A data retornada deve representar o instante de abertura no fuso `-03:00`.

### Erros

- Placa ausente ou inválida: HTTP 422, `{"erro":"placa_invalida"}`.
- `entrada` inválida: HTTP 422, `{"erro":"entrada_invalida"}`.
- Placa com bilhete aberto: HTTP 409, `{"erro":"bilhete_em_aberto"}`.

### Critérios de aceite

1. Uma placa válida sem bilhete aberto cria um bilhete e retorna HTTP 201.
2. A ausência de `entrada` faz o sistema usar o horário atual.
3. Uma `entrada` válida permite definir o instante de abertura.
4. Uma placa inválida retorna o erro especificado.
5. Uma placa com bilhete aberto não pode abrir outro bilhete.

## 5. UC2 — Encerrar bilhete

**Endpoint:** `POST /bilhetes/{id}/encerramento`

### Sucesso

Retornar HTTP 200 com `id`, `placa`, `entrada`, `saida`, `minutos` e `valor_centavos`.

`saida` deve representar o instante do encerramento. `minutos` deve representar a duração total em minutos, calculada a partir dos instantes de entrada e saída.

### Regras de cobrança

1. Se a duração for menor ou igual a `TOLERANCIA_MINUTOS`, cobrar zero.
2. Se a duração ultrapassar a tolerância, calcular a cobrança sobre a duração integral, sem descontar os minutos gratuitos.
3. Calcular a quantidade de frações de cobrança arredondando a duração para cima em múltiplos de `FRACAO_MINUTOS`.
4. O valor de cada fração é `TARIFA_HORA_CENTAVOS / (60 / FRACAO_MINUTOS)`.
5. Limitar o resultado a `TETO_DIARIO_CENTAVOS`.
6. Retornar o valor como inteiro em centavos.

### Erros

- Bilhete inexistente: HTTP 404, `{"erro":"bilhete_nao_encontrado"}`.
- Bilhete já encerrado: HTTP 409, `{"erro":"bilhete_ja_encerrado"}`.
- Bilhete cancelado: HTTP 409, `{"erro":"bilhete_ja_encerrado"}`.

### Critérios de aceite

1. Uma duração dentro da tolerância gera cobrança zero.
2. Uma duração que ultrapassa a tolerância é cobrada integralmente desde o primeiro minuto.
3. Uma duração exata de uma fração cobra uma fração.
4. Um minuto além de uma fração cobra a fração seguinte.
5. O valor calculado nunca ultrapassa o teto.
6. Um bilhete encerrado não pode ser encerrado novamente.

## 6. UC3 — Listar bilhetes ativos

**Endpoint:** `GET /bilhetes/ativos`

Retornar HTTP 200 com um array contendo somente bilhetes com `status: "aberto"`, ordenados do mais recente para o mais antigo.

### Critérios de aceite

1. Bilhetes encerrados e cancelados não aparecem na lista.
2. Os bilhetes abertos aparecem em ordem decrescente de data de entrada.
3. Se não houver bilhetes abertos, retornar um array vazio.

## 7. UC4 — Relatório diário

**Endpoint:** `GET /relatorios/diario?data=AAAA-MM-DD`

### Entrada

- `data`: obrigatória, no formato `AAAA-MM-DD`.

### Sucesso

Retornar HTTP 200 com `data`, `total_bilhetes`, `faturamento_centavos` e `tempo_medio_minutos`.

O relatório deve considerar os bilhetes encerrados na data consultada. O faturamento corresponde à soma dos valores cobrados desses bilhetes. O tempo médio considera somente esses bilhetes e deve ser arredondado para o inteiro mais próximo, com valores terminados em 0,5 arredondados para cima.

### Erros

Data inválida: HTTP 422, `{"erro":"data_invalida"}`.

### Critérios de aceite

1. O relatório inclui apenas bilhetes encerrados na data informada.
2. O faturamento é retornado em centavos inteiros.
3. A média é calculada a partir das durações dos bilhetes considerados.
4. Uma média de 10,5 minutos resulta em 11 minutos.
5. Uma data inválida retorna o erro especificado.

## 8. UC5 — Cancelar bilhete

**Endpoint:** `POST /bilhetes/{id}/cancelamento`

### Sucesso

Retornar HTTP 200 com o estado `cancelado`. O bilhete não deve receber `saida` ou `valor_centavos`.

### Erros

- Bilhete inexistente: HTTP 404, `{"erro":"bilhete_nao_encontrado"}`.
- Bilhete encerrado ou cancelado: HTTP 409, `{"erro":"bilhete_nao_aberto"}`.

### Critérios de aceite

1. Um bilhete aberto pode ser cancelado.
2. O cancelamento não gera cobrança.
3. O bilhete cancelado não pode ser encerrado nem cancelado novamente.
4. Após o cancelamento, a placa pode abrir um novo bilhete.

## 9. UC6 — Histórico por placa

**Endpoint:** `GET /bilhetes?placa=ABC1D23`

### Entrada

- `placa`: obrigatória, com exatamente sete caracteres alfanuméricos maiúsculos.

### Sucesso

Retornar HTTP 200 com todos os bilhetes da placa, independentemente do estado, do mais recente para o mais antigo.

### Erros

Placa ausente ou inválida: HTTP 422, `{"erro":"placa_invalida"}`.

### Critérios de aceite

1. O histórico inclui bilhetes abertos, encerrados e cancelados.
2. Os bilhetes aparecem do mais recente para o mais antigo.
3. Uma placa sem histórico retorna um array vazio.

## 10. UC7 — Tolerância gratuita

A tolerância é definida por `TOLERANCIA_MINUTOS`.

### Critérios de aceite

1. Duração menor ou igual à tolerância: cobrança zero.
2. Duração superior à tolerância: cobrança calculada desde o primeiro minuto.
3. Se a tolerância for zero, toda duração positiva segue as regras normais de cobrança.

## 11. UC8 — Um bilhete aberto por placa

Ao abrir um bilhete, verificar se a placa já possui bilhete aberto.

### Critérios de aceite

1. Se existir bilhete aberto para a placa, retornar HTTP 409 com `{"erro":"bilhete_em_aberto"}`.
2. Bilhetes encerrados ou cancelados não impedem nova abertura.
3. Formatos inválidos devem ser rejeitados com HTTP 422 antes da verificação de conflitos de negócio.

## 12. Regras gerais de resposta

- Todas as respostas devem seguir o contrato de campos e códigos HTTP definido nesta especificação.
- Erros de validação de formato têm precedência sobre conflitos de negócio.
- Não retornar valores monetários em ponto flutuante.
- Todas as listas devem respeitar a ordenação especificada.
