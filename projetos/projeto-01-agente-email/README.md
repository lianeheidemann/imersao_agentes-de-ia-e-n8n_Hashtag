# Projeto 1 — Agente de IA para atendimento inicial por e-mail

Workflow desenvolvido durante a Aula 1 da Imersão Agentes de IA e n8n e adaptado para um cenário próprio de atendimento inicial de clientes de web design.

## Objetivo

Automatizar a triagem inicial de mensagens recebidas por Gmail, identificar solicitações relevantes e produzir uma resposta contextualizada com um agente de IA, preservando decisões comerciais para análise humana.

## Fluxo

1. O **Gmail Trigger** monitora novas mensagens.
2. O node **If** aplica uma condição de filtragem (na versão publicada, uma condição de
   exemplo — verifica se o endereço do remetente contém "gmail" — que deve ser substituída
   pela regra de negócio real antes de qualquer uso além de demonstração, ex.: domínio do
   cliente, assunto ou rótulo do Gmail).
3. O **AI Agent** interpreta o conteúdo do e-mail.
4. O **Google Gemini Chat Model** gera a resposta conforme as regras do prompt.
5. O **Simple Memory** usa o `threadId` do Gmail para separar o contexto de cada conversa.
6. O node **Reply to a message** prepara o envio da resposta na mesma conversa.

## Arquivos

```text
projeto-01-agente-email/
├── README.md
├── prompt/
│   └── prompt-do-agente.txt
└── workflow/
    └── agente-ia-resposta-emails-n8n.json
```

- `workflow/agente-ia-resposta-emails-n8n.json`: exportação sanitizada e desativada do workflow.
- `prompt/prompt-do-agente.txt`: prompt autoral utilizado pelo agente.

## Segurança da versão publicada

A versão deste repositório foi revisada antes da publicação. Foram removidos:

- referências internas de credenciais do Gmail e do Gemini;
- nomes de contas configuradas no n8n;
- e-mails pessoais;
- cabeçalhos e conteúdo de mensagens reais;
- `pinData` de testes;
- identificadores da instância, versão e workflow;
- `webhookId`;
- chave de sessão fixa da memória.

O workflow está publicado com `"active": false`. Após a importação, cada pessoa deve configurar as próprias credenciais diretamente no n8n.

## Importação

1. Inicie uma instância local do n8n.
2. Importe `workflow/agente-ia-resposta-emails-n8n.json`.
3. Configure as credenciais dos nodes Gmail e Gemini.
4. Revise os filtros do node **If**.
5. Execute testes com mensagens fictícias.
6. Ative o workflow somente depois de validar todas as saídas.

Consulte o guia técnico em [`../../docs/abrir-projeto-n8n-com-docker.md`](../../docs/abrir-projeto-n8n-com-docker.md).

## Limitações e melhorias futuras

A versão atual responde automaticamente quando a condição do fluxo é atendida. Para uso real, recomenda-se adicionar uma etapa de aprovação humana ou criação de rascunho antes do envio. Também podem ser adicionados tratamento de erros, logs estruturados, testes com diferentes tipos de e-mail e proteção adicional contra respostas indevidas.

## Autoria e contexto acadêmico

A estrutura inicial foi construída acompanhando a Aula 1 da imersão. O cenário de web design, o prompt, as regras de segurança, os limites de autoridade e as adaptações do workflow foram desenvolvidos para este projeto.

Os materiais didáticos do curso não são reproduzidos neste repositório. O acesso controlado aos materiais de apoio está disponível nesta pasta do Google Drive:

<https://drive.google.com/drive/folders/1k5Efky38YB1H9qrVbMEbBb3o1iOBUFVf?usp=drive_link>
