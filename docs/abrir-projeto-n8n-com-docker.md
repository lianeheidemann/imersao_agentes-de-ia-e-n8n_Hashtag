# Como abrir um projeto do n8n usando Docker Desktop

Este guia explica como acessar o n8n executado localmente pelo Docker Desktop e importar um workflow criado em outra instalação do n8n.

> O n8n aberto pelo Docker é uma instalação separada do n8n Cloud ou de qualquer outra instância usada anteriormente. Por isso, os workflows antigos não aparecem automaticamente: é necessário exportá-los e importá-los.

## 1. Confirmar que o container está em execução

Abra o **Docker Desktop** e acesse **Containers**.

O container do n8n deve aparecer com o status **Running**. Na tela de logs, procure uma mensagem semelhante a:

```text
Editor is now accessible via:
http://localhost:5678
```

No exemplo deste projeto, o container foi criado com um nome automático, como `nervous_aryabhata`. O nome pode ser diferente em outro computador e não interfere no funcionamento.

## 2. Abrir o editor do n8n

Com o container em execução, abra o navegador e acesse:

```text
http://localhost:5678
```

Também é possível clicar diretamente no link exibido nos logs do Docker Desktop.

Na primeira utilização, o n8n pode solicitar a criação da conta administradora local. Essa conta pertence apenas à instalação executada no seu computador.

## 3. Entender por que o projeto não apareceu automaticamente

Quando os logs mostram algo semelhante a:

```text
Processed 0 draft workflows, 0 published workflows
```

isso significa que a instalação local ainda não possui workflows salvos.

O workflow `Hashtag_Aula_1`, criado em outra instalação do n8n, precisa ser exportado para um arquivo `.json` e depois importado na instalação local.

## 4. Exportar o workflow da instalação original

Na instalação em que o projeto está salvo:

1. Abra o workflow `Hashtag_Aula_1`.
2. Clique no menu de três pontos, no canto superior direito.
3. Selecione **Download**, **Export** ou **Download workflow**, conforme a versão do n8n.
4. Salve o arquivo no computador.
5. Use um nome claro, por exemplo:

```text
agente-ia-resposta-emails-n8n.json
```

Antes de publicar esse arquivo no GitHub, revise o conteúdo para garantir que não existam e-mails pessoais, tokens, chaves de API, dados de clientes ou outras informações privadas.

## 5. Importar o workflow no n8n local

Acesse `http://localhost:5678` e siga estes passos:

1. Entre no espaço **Personal**.
2. Crie um workflow novo ou abra o menu de opções.
3. Selecione **Import from File** ou **Importar de arquivo**.
4. Escolha o arquivo JSON exportado.
5. Aguarde o workflow aparecer no editor.
6. Salve o workflow.

Em algumas versões, também é possível arrastar o arquivo `.json` diretamente para o editor do n8n.

## 6. Configurar novamente as credenciais

As credenciais da instalação anterior não ficam disponíveis automaticamente na instalação local. Depois da importação, abra os nodes que dependem de autenticação e selecione ou crie novas credenciais.

Neste projeto, revise principalmente:

- **Gmail Trigger**: credencial OAuth do Gmail;
- **Google Gemini Chat Model**: chave ou credencial do Gemini;
- **Reply to a message**: credencial do Gmail usada para responder à mensagem.

Nunca escreva senhas ou chaves diretamente no workflow. Use o sistema de credenciais do próprio n8n.

### Gmail e endereço de redirecionamento

Ao criar uma credencial OAuth do Gmail, o n8n mostra uma URL de redirecionamento. Caso você use credenciais próprias do Google Cloud, essa URL deve ser cadastrada como URI de redirecionamento autorizada no projeto do Google.

Para uma instalação local, ela normalmente começa com:

```text
http://localhost:5678/rest/oauth2-credential/callback
```

Use sempre o endereço exibido na tela da credencial do seu n8n, pois ele é a referência correta para a sua instalação.

## 7. Testar o workflow com segurança

Antes de publicar ou ativar o workflow:

1. Confirme que todos os nodes estão conectados corretamente.
2. Use uma conta e mensagens de teste.
3. Execute manualmente o workflow.
4. Confira a saída do node **AI Agent**.
5. Verifique se a resposta gerada está correta antes de permitir o envio automático.
6. Somente depois dos testes, publique ou ative o workflow.

Para um projeto de portfólio, é mais seguro configurar o fluxo para criar um **rascunho** ou exigir revisão humana antes de enviar uma resposta real.

## 8. Evitar a perda dos workflows ao excluir o container

Os dados do n8n devem ser armazenados fora da camada temporária do container. Para isso, use um volume persistente ligado ao diretório:

```text
/home/node/.n8n
```

No Docker Desktop, abra o container e verifique as abas **Inspect**, **Bind mounts** ou **Volumes**. Procure um mapeamento para `/home/node/.n8n`.

Se não existir um volume, não exclua o container antes de exportar os workflows. A exclusão de um container sem armazenamento persistente pode remover os dados salvos nele.

## 9. Forma recomendada de executar com volume persistente

No PowerShell, crie primeiro um volume:

```powershell
docker volume create n8n_data
```

Depois, execute o n8n usando esse volume:

```powershell
docker run --name n8n-local `
  -p 5678:5678 `
  -e GENERIC_TIMEZONE=America/Belem `
  -e TZ=America/Belem `
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true `
  -v n8n_data:/home/node/.n8n `
  docker.n8n.io/n8nio/n8n
```

Esse comando:

- cria um container chamado `n8n-local`;
- disponibiliza o editor na porta `5678`;
- configura o fuso horário de Belém;
- salva workflows, configurações e credenciais criptografadas no volume `n8n_data`.

Para executar o container em segundo plano, acrescente `-d` ao comando:

```powershell
docker run -d --name n8n-local `
  -p 5678:5678 `
  -e GENERIC_TIMEZONE=America/Belem `
  -e TZ=America/Belem `
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true `
  -v n8n_data:/home/node/.n8n `
  docker.n8n.io/n8nio/n8n
```

Depois da criação inicial, use estes comandos:

```powershell
# Iniciar

docker start n8n-local

# Parar

docker stop n8n-local

# Ver os logs

docker logs -f n8n-local
```

## 10. Solução de problemas

### O endereço `localhost:5678` não abre

Confira se:

- o Docker Desktop está aberto;
- o status do container é **Running**;
- a porta `5678` está publicada;
- outro programa não está usando a mesma porta.

No PowerShell, execute:

```powershell
docker ps
```

Na coluna **PORTS**, deve aparecer algo semelhante a:

```text
0.0.0.0:5678->5678/tcp
```

### O workflow foi importado, mas os nodes mostram erro

Abra cada node com erro e configure novamente as credenciais. Os segredos da instalação original não são transferidos como credenciais funcionais para a nova instalação.

### O workflow desapareceu depois de recriar o container

Provavelmente o container foi criado sem volume persistente. Recrie-o usando `n8n_data:/home/node/.n8n` e importe novamente o arquivo JSON salvo.

### A porta 5678 já está ocupada

Use outra porta no computador, mantendo a porta interna do n8n:

```powershell
docker run -d --name n8n-local `
  -p 5679:5678 `
  -v n8n_data:/home/node/.n8n `
  docker.n8n.io/n8nio/n8n
```

Nesse caso, acesse:

```text
http://localhost:5679
```

## 11. Checklist final

- [ ] Docker Desktop aberto;
- [ ] container do n8n com status **Running**;
- [ ] editor acessível pelo navegador;
- [ ] workflow exportado em formato JSON;
- [ ] workflow importado no n8n local;
- [ ] credenciais do Gmail configuradas novamente;
- [ ] credencial do Gemini configurada novamente;
- [ ] teste realizado com dados fictícios;
- [ ] volume persistente configurado;
- [ ] nenhuma senha ou chave publicada no GitHub.

## Referências

- Documentação oficial do n8n sobre instalação com Docker: <https://docs.n8n.io/hosting/installation/docker/>
- Documentação oficial sobre importação e exportação de workflows: <https://docs.n8n.io/workflows/export-import/>
- Documentação oficial sobre segurança: <https://docs.n8n.io/hosting/securing/>
