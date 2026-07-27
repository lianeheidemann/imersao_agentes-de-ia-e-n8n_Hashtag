# Execução local do n8n com Docker Desktop

Este documento descreve uma configuração local reproduzível do n8n no Windows com Docker Desktop, persistência de dados, importação de workflows e práticas mínimas de segurança.

## 1. Arquitetura local

A configuração utiliza:

- Docker Desktop com backend WSL 2;
- imagem oficial `docker.n8n.io/n8nio/n8n`;
- porta HTTP local `5678`;
- volume nomeado para persistir o diretório `/home/node/.n8n`;
- chave de criptografia definida fora do repositório;
- fuso horário `America/Belem`.

O diretório `/home/node/.n8n` contém dados importantes da instância, incluindo configuração, banco local e credenciais criptografadas. Executar o container sem volume persistente pode causar perda de dados ao removê-lo.

## 2. Pré-requisitos

No PowerShell, confirme as instalações:

```powershell
docker --version
docker compose version
```

Confirme também que o Docker Desktop está com o engine ativo.

## 3. Estrutura local recomendada

Crie uma pasta fora do repositório para armazenar a configuração da instância:

```text
n8n-local/
├── compose.yaml
└── .env
```

O arquivo `.env` contém segredos locais e não deve ser enviado ao GitHub.

## 4. Configuração com Docker Compose

Crie `compose.yaml`:

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n-local
    restart: unless-stopped
    ports:
      - "127.0.0.1:5678:5678"
    environment:
      TZ: America/Belem
      GENERIC_TIMEZONE: America/Belem
      N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS: "true"
      N8N_ENCRYPTION_KEY: ${N8N_ENCRYPTION_KEY}
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

A vinculação `127.0.0.1:5678:5678` limita o acesso à máquina local. Para uma implantação acessível pela internet, são necessários HTTPS, proxy reverso, domínio e configurações adicionais de segurança.

## 5. Chave de criptografia

Gere uma chave longa no PowerShell:

```powershell
$bytes = New-Object byte[] 32
[Security.Cryptography.RandomNumberGenerator]::Fill($bytes)
[Convert]::ToBase64String($bytes)
```

Crie `.env`:

```dotenv
N8N_ENCRYPTION_KEY=SUBSTITUA_PELA_CHAVE_GERADA
```

Guarde essa chave com segurança. Alterá-la ou perdê-la pode impedir a leitura de credenciais já criptografadas pela instância.

## 6. Inicialização

No diretório que contém `compose.yaml`, execute:

```powershell
docker compose up -d
docker compose ps
docker compose logs -f n8n
```

Quando o serviço estiver pronto, acesse:

```text
http://localhost:5678
```

Na primeira execução, conclua o cadastro da conta proprietária local.

## 7. Operações do ciclo de vida

```powershell
# Iniciar ou recriar conforme a configuração
docker compose up -d

# Parar sem remover container e volume
docker compose stop

# Reiniciar
docker compose restart n8n

# Ver logs
docker compose logs -f n8n

# Encerrar e remover o container, preservando o volume
docker compose down
```

Não execute `docker compose down -v` sem backup. A opção `-v` remove o volume nomeado e pode apagar workflows, configurações e credenciais da instância.

## 8. Identificar conflitos entre containers

Somente um processo pode publicar a mesma porta local. Liste os containers:

```powershell
docker ps -a --filter ancestor=n8nio/n8n:latest
docker ps -a --filter ancestor=docker.n8n.io/n8nio/n8n
```

Verifique portas:

```powershell
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

Se outro container estiver usando `5678`, pare-o:

```powershell
docker stop NOME_DO_CONTAINER
```

Como alternativa, altere a porta externa:

```yaml
ports:
  - "127.0.0.1:5679:5678"
```

Nesse caso, use `http://localhost:5679`.

## 9. Importar o Projeto 1

O workflow sanitizado está em:

```text
projetos/projeto-01-agente-email/workflow/agente-ia-resposta-emails-n8n.json
```

No editor do n8n:

1. Abra um projeto ou workflow vazio.
2. Use **Import from File**.
3. Selecione o arquivo JSON.
4. Salve o workflow.
5. Configure novas credenciais nos nodes:
   - `Gmail Trigger`;
   - `Google Gemini Chat Model`;
   - `Reply to a message`.
6. Revise a condição do node `If`.
7. Execute testes manuais antes da ativação.

A versão publicada está desativada e não contém credenciais funcionais.

## 10. Credenciais e OAuth do Gmail

As credenciais pertencem à instância do n8n e não são transportadas como segredos funcionais no workflow sanitizado. Configure uma nova credencial OAuth no node do Gmail.

Quando o Google Cloud solicitar uma URI de redirecionamento, copie exatamente a URL exibida pelo n8n. Em uma instalação local padrão, o formato costuma ser:

```text
http://localhost:5678/rest/oauth2-credential/callback
```

Não publique `client_secret`, tokens OAuth, chaves do Gemini ou o arquivo `.env`.

## 11. Testes seguros

Antes de ativar o workflow:

- utilize contas e mensagens de teste;
- confirme que o filtro não processa mensagens indevidas;
- valide a saída HTML do agente;
- teste mensagens incompletas, spam e tentativas de manipulação do prompt;
- verifique se o contexto da memória está separado por `threadId`;
- prefira criação de rascunho ou aprovação humana antes do envio automático em ambientes reais.

## 12. Backup

Liste o volume:

```powershell
docker volume ls
docker volume inspect n8n-local_n8n_data
```

Uma forma de backup do volume para o diretório atual é:

```powershell
docker run --rm `
  -v n8n-local_n8n_data:/data `
  -v "${PWD}:/backup" `
  alpine `
  tar czf /backup/n8n-data-backup.tar.gz -C /data .
```

O nome real do volume pode variar conforme o nome da pasta do projeto Compose. Confirme-o com `docker volume ls`.

Além do backup do volume, exporte periodicamente os workflows em JSON. Não publique backups porque eles podem conter credenciais criptografadas, dados de execução e informações pessoais.

## 13. Atualização da imagem

```powershell
docker compose pull
docker compose up -d
docker image prune
```

Antes de atualizar, faça backup do volume e consulte as notas de versão, especialmente em atualizações maiores.

## 14. Diagnóstico

### Editor não abre

```powershell
docker compose ps
docker compose logs --tail 200 n8n
Test-NetConnection localhost -Port 5678
```

### Container reinicia continuamente

```powershell
docker inspect n8n-local
docker compose logs --tail 300 n8n
```

Verifique sintaxe do `.env`, permissões do volume, variáveis obrigatórias e conflitos de porta.

### Workflow importado apresenta nodes desconectados de credenciais

Esse comportamento é esperado na versão pública. Abra cada node e selecione credenciais locais válidas.

### Workflow desapareceu

Confirme se o container atual utiliza o mesmo volume persistente. Containers diferentes podem apontar para volumes diferentes e, por isso, apresentar conjuntos distintos de workflows.

## 15. Checklist

- [ ] Docker Desktop ativo;
- [ ] apenas um container usando a porta escolhida;
- [ ] volume persistente montado em `/home/node/.n8n`;
- [ ] `N8N_ENCRYPTION_KEY` definida em `.env`;
- [ ] `.env` fora do Git;
- [ ] workflow importado;
- [ ] credenciais configuradas localmente;
- [ ] testes realizados com dados fictícios;
- [ ] workflow ativado somente após validação;
- [ ] backup criado antes de atualizações.

## Referências oficiais

- <https://docs.n8n.io/hosting/installation/docker/>
- <https://docs.n8n.io/workflows/export-import/>
- <https://docs.n8n.io/hosting/configuration/configuration-examples/encryption-key/>
- <https://docs.n8n.io/hosting/configuration/configuration-examples/time-zone/>
- <https://docs.n8n.io/hosting/securing/>
