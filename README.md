# Samba Connect

Script para conectar e montar diretórios hospedados em um servidor Samba utilizando o protocolo CIFS.

## Sumário

- [Descrição](#descrição)
- [Requisitos](#requisitos)
- [Instalação das dependências](#instalação-das-dependências)
- [Instalação do script](#instalação-do-script)
- [Configuração do arquivo de credenciais](#configuração-do-arquivo-de-credenciais)
- [Uso](#uso)
- [Parâmetros](#parâmetros)
- [Exemplos](#exemplos)
- [Utilizando um endereço DNS ou DDNS](#utilizando-um-endereço-dns-ou-ddns)
- [Permissões do sudo](#permissões-do-sudo)
- [Desmontando um compartilhamento](#desmontando-um-compartilhamento)
- [Solução de problemas](#solução-de-problemas)
- [Opções de montagem utilizadas](#opções-de-montagem-utilizadas)
- [Segurança](#segurança)
- [Referências](#referências)
- [Licença](#licença)

## Descrição

O **Samba Connect** é um script Bash utilizado para montar diretórios compartilhados em um servidor Samba por meio do protocolo CIFS.

O script:

- Cria automaticamente o diretório de montagem em `/mnt`;
- Utiliza um arquivo separado para armazenar as credenciais;
- Verifica se o diretório já está montado;
- Define permissões para arquivos e diretórios;
- Permite informar o servidor, o compartilhamento e o grupo de montagem;
- Utiliza SMB versão `3.1.1`;
- Exibe mensagens informando o resultado da operação.

## Requisitos

- Linux;
- Bash;
- Acesso a um servidor Samba;
- Permissão para executar o comando `mount`;
- Pacote `cifs-utils`.

## Instalação das dependências

### Debian, Ubuntu e derivados

```bash
sudo apt update
sudo apt install cifs-utils
```

### Fedora, RHEL e derivados

```bash
sudo dnf install cifs-utils
```

### Arch Linux e derivados

```bash
sudo pacman -S cifs-utils
```

## Instalação do script

Clone o repositório do projeto:

```bash
git clone https://github.com/MrWesleyR/SambaConnectCLI
```

Acesse o diretório do projeto:

```bash
cd <DIRETORIO_DO_REPOSITORIO>
```

Dê permissão de execução ao script:

```bash
chmod +x smbConnect
```

Opcionalmente, mova o script para um diretório disponível no PATH:

```bash
mkdir -p "$HOME/.local/bin"
mv smbConnect "$HOME/.local/bin/"
```

Verifique se o diretório `~/.local/bin` está disponível no seu PATH:

```bash
echo "$PATH"
```

Caso não esteja, adicione a seguinte linha ao arquivo `~/.bashrc`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Depois, recarregue as configurações do shell:

```bash
source "$HOME/.bashrc"
```

Se você utiliza o Zsh, adicione a linha ao arquivo `~/.zshrc` e execute:

```bash
source "$HOME/.zshrc"
```

## Configuração do arquivo de credenciais

Crie o diretório onde o arquivo de credenciais será armazenado:

```bash
mkdir -p "$HOME/.smb"
```

Crie um arquivo de credenciais:

```bash
nano "$HOME/.smb/smbCredentials"
```

Adicione as informações do usuário Samba:

```ini
username=SEU_USUARIO
password=SUA_SENHA
```

Caso o servidor utilize um domínio, adicione também:

```ini
domain=SEU_DOMINIO
```

Exemplo completo:

```ini
username=user
password=minha_senha
domain=WORKGROUP
```

Proteja o arquivo para que somente o proprietário tenha acesso:

```bash
chmod 600 "$HOME/.smb/smbCredentials"
```

Ajuste o proprietário do arquivo:

```bash
chown "$USER":"$(id -gn)" "$HOME/.smb/smbCredentials"
```

O caminho final do arquivo deverá ser semelhante a:

```text
/home/seu_usuario/.smb/smbCredentials
```

> [!WARNING] Nunca versionei ou publique arquivos que contenham senhas reais.

## Uso

O script deve ser executado com os seguintes parâmetros:

```bash
smbConnect <pasta> <credenciais> <host> [grupo]
```

Formato completo:

```bash
smbConnect NOME_DO_COMPARTILHAMENTO ARQUIVO_DE_CREDENCIAIS ULTIMO_OCTETO [GRUPO]
```

O servidor é montado utilizando o formato:

```text
//192.168.1.<ULTIMO_OCTETO>/<PASTA>
```

O ponto de montagem será criado em:

```text
/mnt/<PASTA>
```

## Parâmetros

| Parâmetro | Obrigatório | Descrição |
|:-----------:|:--------------:|:----------:|
| pasta | Sim | Nome do compartilhamento existente no servidor Samba |
| credenciais | Sim | Nome do arquivo de credenciais localizado em `~/.smb/` |
| host | Sim | Último octeto do endereço IP do servidor |
| grupo | Não | GID ou grupo utilizado na montagem. O padrão é `1000` |

## Exemplos

### Montagem básica

Para montar o compartilhamento `documentos` localizado no servidor `192.168.1.100`:

```bash
smbConnect documentos smbCredentials 100
```

Nesse caso, o script tentará montar:

```text
//192.168.1.100/documentos
```

No diretório:

```text
/mnt/documentos
```

### Informando um grupo específico

```bash
smbConnect documentos smbCredentials 100 1000
```

### Executando diretamente do diretório do projeto

Se o script não estiver instalado no PATH:

```bash
./smbConnect documentos smbCredentials 100
```

Ou:

```bash
bash ./smbConnect documentos smbCredentials 100
```

## Utilizando um endereço DNS ou DDNS

A versão atual do script monta servidores no formato `192.168.1.<host>`. Para utilizar um domínio ou endereço DDNS, altere a variável `server` no script.

Exemplo:

```bash
server="//meu-servidor.example.com"
```

Depois, a montagem poderá ser executada usando:

```bash
smbConnect documentos smbCredentials qualquer_valor
```

Para aceitar diretamente um endereço IP completo ou um domínio como parâmetro, o script precisará ser adaptado.

## Permissões do **sudo**

A montagem de sistemas de arquivos normalmente exige privilégios administrativos.

Verifique as permissões disponíveis para o seu usuário:

```bash
sudo -l
```

Se for necessário utilizar sudo, execute:

```bash
sudo smbConnect documentos smbCredentials
```

Se o usuário já possuir permissão para executar o script ou o comando de montagem sem privilégios adicionais, execute normalmente:

```bash
smbConnect documentos smbCredentials 100
```

Para alterar regras do sudo, utilize o comando:

```bash
sudo visudo
```

Evite editar diretamente o arquivo `/etc/sudoers`. O comando `visudo` verifica erros de sintaxe antes de salvar as alterações.

## Desmontando um compartilhamento

Para desmontar o diretório:

```bash
sudo umount /mnt/documentos
```

Também é possível utilizar o comando `umount` com o caminho do compartilhamento:

```bash
sudo umount /mnt/<PASTA>
```

Para verificar os compartilhamentos montados:

```bash
findmnt
```

Para verificar especificamente um diretório:

```bash
findmnt /mnt/documentos
```

## Solução de problemas

### Arquivo de credenciais não encontrado

Verifique se o arquivo existe em `~/.smb/`:

```bash
ls -la "$HOME/.smb"
```

Confirme se o nome informado no comando está correto:

```bash
smbConnect documentos smbCredentials 100
```

Nesse exemplo, o arquivo esperado é:

```text
~/.smb/smbCredentials
```

### Permissão negada

Verifique as permissões do arquivo de credenciais:

```bash
ls -l "$HOME/.smb/smbCredentials"
```

O resultado esperado deve restringir o acesso ao proprietário:

```text
-rw-------
```

Corrija as permissões, se necessário:

```bash
chmod 600 "$HOME/.smb/smbCredentials"
```

### O diretório já está montado

Verifique os pontos de montagem ativos:

```bash
findmnt
```

Se necessário, desmonte o diretório:

```bash
sudo umount /mnt/documentos
```

### Falha ao conectar ao servidor

Verifique:

- Se o servidor está ligado;
- Se o endereço IP está correto;
- Se o compartilhamento existe;
- Se o servidor está acessível pela rede;
- Se as credenciais estão corretas;
- Se as portas do Samba não estão bloqueadas pelo firewall.

Teste a conectividade com o servidor:

```bash
ping 192.168.1.100
```

Também é possível listar os compartilhamentos disponíveis:

```bash
smbclient -L //192.168.1.100 -U SEU_USUARIO
```

### Verificando os logs do sistema

Em sistemas que utilizam systemd, consulte as mensagens do kernel:

```bash
dmesg | tail -n 50
```

Ou:

```bash
journalctl -k -n 50
```

## Opções de montagem utilizadas

O script utiliza algumas opções específicas do CIFS:

| Opção | Descrição |
|:-----------|:----------|
| `credentials` | Define o arquivo que contém usuário e senha |
| `dir_mode=0775` | Define as permissões padrão dos diretórios |
| `file_mode=0775` | Define as permissões padrão dos arquivos |
| `uid=1000` | Define o usuário proprietário dos arquivos |
| `gid` | Define o grupo proprietário dos arquivos |
| `forceuid` | Força o uso do UID informado |
| `forcegid` | Força o uso do GID informado |
| `vers=3.1.1` | Define a versão do protocolo SMB |
| `cache=none` | Desativa o cache de dados do cliente |
| `use=inode` | Utiliza os números de inode fornecidos pelo servidor |
| `noperm` | Desativa a verificação local de permissões |
| `actimeo=3` | Define o tempo de cache dos atributos dos arquivos |

## Segurança

- Não armazene senhas diretamente no script.
- Não publique o arquivo de credenciais no GitHub ou em outro repositório público.
- Utilize permissões `600` no arquivo de credenciais.
- Evite conceder permissões de sudo mais amplas do que o necessário.
- Utilize usuários com permissões limitadas no servidor Samba.
- Revise o arquivo `.gitignore` antes de realizar um commit.

Exemplo de `.gitignore`:

```gitignore
*.smb
*.cred
credentials
smbCredentials
.smb/
```

## Referências

- [mount.cifs — Linux manual page](https://man7.org/linux/man-pages/man8/mount.cifs.8.html)
- [Using sudo to delegate permissions](https://www.sudo.ws/doc.html#DelegatingPermissions)
- [Documentação do Samba](https://www.samba.org/samba/docs/)

## Licença

Distribua este projeto sob a licença de sua preferência.

Exemplo:

```text
MIT License
```

Consulte o arquivo LICENSE para obter os termos completos da licença.
