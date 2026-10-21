# Constitution — Zona Azul Digital

## 1. Objetivo

Estabelecer regras obrigatórias para a implementação da API REST de bilhetes de estacionamento rotativo. Estas regras se aplicam a todos os endpoints e componentes da aplicação.

## 2. Regras obrigatórias

### 2.1. Contrato da API

- Implementar os oito casos de uso definidos no `spec.md`, respeitando os métodos HTTP, caminhos, parâmetros, códigos de status, nomes de campos e corpos de resposta especificados.
- Não substituir os nomes dos campos ou códigos de erro definidos na especificação por alternativas.
- Toda resposta de erro deve utilizar o formato JSON definido para o respectivo caso.

### 2.2. Valores monetários

- Representar todos os valores monetários em centavos inteiros.
- Nunca retornar valores monetários em ponto flutuante ou em reais decimais.
- Calcular o valor de cada fração a partir de `TARIFA_HORA_CENTAVOS` e `FRACAO_MINUTOS`.
- Nunca cobrar acima de `TETO_DIARIO_CENTAVOS` por bilhete.

### 2.3. Datas e horários

- Aceitar datas e horários no formato ISO-8601 com fuso horário quando exigido pelo contrato.
- Utilizar o fuso `-03:00` nas datas e horários retornados pela API.
- Quando `entrada` não for informada na abertura do bilhete, utilizar o instante atual.
- Permitir o fornecimento de `entrada` para tornar os cálculos de duração reproduzíveis nos testes.

### 2.4. Validação e erros

- Validar os formatos dos dados antes de verificar conflitos de regras de negócio.
- Retornar HTTP 422 para entradas inválidas, HTTP 404 para bilhetes inexistentes e HTTP 409 para conflitos de estado ou de negócio, conforme o contrato.
- Não permitir que uma placa tenha mais de um bilhete aberto simultaneamente.

### 2.5. Estados e integridade

- Um bilhete aberto pode ser encerrado ou cancelado.
- Um bilhete encerrado não pode ser encerrado novamente.
- Um bilhete cancelado não pode ser encerrado nem cancelado novamente.
- O cancelamento não gera cobrança e não deve adicionar os campos `saida` ou `valor_centavos` ao bilhete cancelado.
- Após encerrar ou cancelar um bilhete, permitir uma nova abertura para a mesma placa.

### 2.6. Critérios de implementação

- Manter a lógica de cobrança centralizada para que todos os cálculos sigam as mesmas regras.
- Utilizar os parâmetros de `variante/params.json` como fonte dos valores específicos da prova.
- O serviço deve escutar na porta definida por `PORTA_SERVICO`, sem exigir variáveis de ambiente adicionais.
- Não implementar funcionalidades de back-office, interface gráfica ou outros recursos fora do escopo da API.
