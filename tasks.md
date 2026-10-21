# Tarefas de Implementação — Zona Azul Digital

## T1 — Estrutura e configuração

- Definir a stack e organizar a estrutura da aplicação.
- Configurar a porta a partir de `variante/params.json`.
- Criar o manifesto de dependências e as instruções de execução.

**Conclusão:** o serviço inicia na porta exigida e possui estrutura documentada.

## T2 — Modelo e persistência

- Implementar o modelo de bilhete e seus estados.
- Implementar persistência e geração de identificadores únicos.
- Garantir um único bilhete aberto por placa.

**Conclusão:** bilhetes podem ser armazenados e consultados preservando as regras de estado.

## T3 — Abertura, encerramento e cancelamento

- Implementar os endpoints de abertura, encerramento e cancelamento.
- Implementar validação, cálculo de duração, tolerância, frações e teto.
- Garantir os códigos HTTP e corpos de erro especificados.

**Conclusão:** os cenários de UC1, UC2, UC5, UC7 e UC8 são atendidos.

## T4 — Consultas e relatório

- Implementar listagem de bilhetes ativos.
- Implementar histórico por placa.
- Implementar relatório diário e arredondamento da média.

**Conclusão:** os cenários de UC3, UC4 e UC6 são atendidos.

## T5 — Verificação e entrega

- Executar os testes automatizados.
- Corrigir divergências em relação ao contrato.
- Documentar instalação, execução e testes.
- Verificar dependências e ausência de segredos no repositório.

**Conclusão:** a aplicação está documentada e os testes relevantes passam.
