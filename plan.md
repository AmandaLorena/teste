# Plano Técnico — Zona Azul Digital

## 1. Arquitetura

Implementar uma API REST com separação entre rotas HTTP, regras de negócio e persistência. As rotas devem validar as requisições, delegar operações aos serviços e retornar os códigos HTTP e corpos definidos em `spec.md`.

## 2. Stack e execução

- Escolher uma stack simples, adequada à implementação de uma API REST.
- Incluir manifesto de dependências e instruções para execução.
- O serviço deve escutar na porta definida por `PORTA_SERVICO` em `variante/params.json`.
- Não exigir variáveis de ambiente adicionais.

## 3. Persistência

Utilizar persistência local simples e confiável, adequada ao escopo da aplicação. Os bilhetes devem manter seus identificadores, placa, entrada, estado e, quando aplicável, saída, duração e valor cobrado.

Garantir que a criação de bilhetes preserve a unicidade dos identificadores e que as alterações de estado não permitam dois bilhetes abertos para a mesma placa.

## 4. Datas e relógio

Centralizar a obtenção do instante atual em um componente de relógio. Aceitar a entrada opcional definida no contrato para permitir testes reproduzíveis. Interpretar e retornar datas conforme o fuso `-03:00`.

## 5. Cálculo de cobrança

Centralizar o cálculo em uma única regra de negócio:

1. Calcular a duração em minutos.
2. Se a duração for menor ou igual à tolerância, cobrar zero.
3. Se ultrapassar a tolerância, calcular sobre a duração integral.
4. Arredondar a quantidade de frações para cima.
5. Calcular cada fração a partir da tarifa horária e do tamanho da fração.
6. Aplicar o teto por bilhete.
7. Retornar o resultado em centavos inteiros.

## 6. Qualidade e entrega

- Criar testes automatizados para os casos de uso e casos de borda.
- Documentar instalação, execução e testes no README.
- Manter dependências necessárias declaradas no manifesto.
- Não incluir segredos nem arquivos gerados desnecessários no repositório.
