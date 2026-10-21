---

## Índice da especificação

Este repositório contém a especificação funcional e técnica da API Zona Azul Digital. Leia os documentos abaixo para compreender os requisitos antes de gerar a implementação.

### Documentos de especificação

1. [`constitution.md`](constitution.md) — regras obrigatórias de implementação, validação, datas, valores monetários e integridade dos bilhetes.
2. [`spec.md`](spec.md) — requisitos funcionais, endpoints REST, respostas HTTP, regras de negócio e critérios de aceite dos casos de uso UC1 a UC8.
3. [`plan.md`](plan.md) — decisões técnicas, arquitetura, persistência, relógio e cálculo de cobrança.
4. [`tests.md`](tests.md) — casos de teste, incluindo cenários de borda, erros e limites de cobrança.
5. [`tasks.md`](tasks.md) — decomposição do trabalho em tarefas de implementação e critérios de conclusão.

### Orientação para leitura

Leia todos os documentos vinculados antes de iniciar a implementação. Em caso de divergência, siga o contrato definido em `contrato.json` e os parâmetros específicos de `variante/params.json`. Não invente comportamentos que não estejam especificados.

O entregável desta atividade é a especificação em Markdown; os documentos devem orientar a geração do código sem conter a implementação completa.
