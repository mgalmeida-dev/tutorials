# GitHub pelo terminal do VS Code com o GitHub CLI

Enquanto organizava meus projetos de estudo no VS Code, surgiu uma necessidade prática: executar os comandos de Git e realizar tarefas do GitHub no mesmo ambiente, reduzindo a troca entre o editor e as abas do navegador.

Foi assim que comecei a usar o **GitHub CLI**, chamado pelo comando `gh`. Neste tutorial, reuni o processo de instalação e autenticação, a explicação dos comandos e alguns exemplos para começar.

## Conteúdo

- [Git, GitHub e GitHub CLI](#git-github-e-github-cli)
- [Ambiente e pré-requisitos](#ambiente-e-pré-requisitos)
- [Instalação explicada passo a passo](#instalação-explicada-passo-a-passo)
- [Autenticação no GitHub](#autenticação-no-github)
- [Primeiros comandos no projeto](#primeiros-comandos-no-projeto)
- [Exemplo: atualizar o README e abrir um pull request](#exemplo-atualizar-o-readme-e-abrir-um-pull-request)
- [Como consultar ajuda](#como-consultar-ajuda)
- [Referências oficiais](#referências-oficiais)

## Git, GitHub e GitHub CLI

O **Git** é o sistema de controle de versão: registra o histórico das alterações de um projeto e permite trabalhar com branches.

O **GitHub** é a plataforma que hospeda repositórios Git e oferece recursos como pull requests, issues e automações.

O **GitHub CLI** permite acessar funcionalidades do GitHub pela linha de comando. CLI significa *Command-Line Interface*: interface de linha de comando.

As ferramentas se complementam:

| Objetivo                                          | Comando |
| --- | --- |
| Ver quais arquivos foram alterados                | `git status` |
| Preparar alterações para um commit                | `git add` |
| Registrar alterações no histórico local           | `git commit` |
| Enviar commits para um repositório remoto         | `git push` |
| Consultar pull requests no GitHub                 | `gh pr list` |
| Criar um pull request no GitHub                   | `gh pr create` |
| Alterar a descrição de um repositório no GitHub   | `gh repo edit` |

Continuo usando o Git para versionar o projeto e utilizo o GitHub CLI para acessar recursos da plataforma.

## Ambiente e pré-requisitos

**A instalação deste tutorial é destinada a Ubuntu/Debian com APT, inclusive quando executados pelo WSL no Windows.**

O terminal integrado do VS Code pode executar ambientes diferentes, como PowerShell, Git Bash ou uma distribuição Linux pelo WSL. Ter o VS Code no Windows não significa que todo terminal aberto nele seja Linux.

Para acompanhar:

- Utilize um ambiente Ubuntu/Debian com APT e acesso a `sudo`.
- Tenha uma conta no GitHub para a etapa de autenticação.
- Tenha o Git instalado para acompanhar os exemplos de versionamento.

No VS Code, abra **Terminal → Novo Terminal** e selecione o ambiente Linux correspondente.

Confira a instalação do Git:

```bash
git --version
```

## Instalação explicada passo a passo

A sequência configura a fonte oficial de pacotes do GitHub CLI e instala a ferramenta. Execute uma etapa por vez e confira o resultado antes de avançar. Se uma etapa apresentar erro, resolva-o antes de continuar.

### 1. Atualizar a lista de pacotes

```bash
sudo apt update
```

| Trecho | Função |
| --- | --- |
| `sudo` | Executa o comando com privilégios administrativos, conforme as permissões da conta. |
| `apt` | Gerencia pacotes em sistemas como Ubuntu e Debian. |
| `update` | Atualiza o catálogo local de pacotes disponíveis nas fontes configuradas. |

**Esse comando atualiza a lista de pacotes; ele não atualiza automaticamente todos os programas instalados.**

Se o terminal pedir sua senha, é normal que nenhum caractere apareça enquanto você digita.

### 2. Instalar o curl

```bash
sudo apt install -y curl
```

O `curl` permite transferir dados por URLs. Aqui, será usado para baixar um arquivo.

- `install`: solicita a instalação do pacote.
- `-y`: responde “sim” automaticamente às confirmações usuais do APT.
- `curl`: identifica o pacote que será instalado.

Se já estiver instalado, o APT verificará a versão disponível e informará o resultado.

### 3. Preparar a pasta das chaves

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
```

Essa linha prepara o diretório onde ficará a chave pública utilizada para verificar as assinaturas da fonte de pacotes.

| Trecho | Função |
| --- | --- |
| `mkdir` | Cria um diretório. |
| `-p` | Cria os diretórios intermediários necessários e aceita que o destino já exista. |
| `-m 755` | Define as permissões do diretório ao criá-lo. |
| `/etc/apt/keyrings` | Indica o caminho do diretório. |

O `755` permite que o proprietário leia, escreva e acesse o diretório; os demais usuários podem ler e acessar. Se o diretório já existir, `mkdir -p -m 755` não redefine suas permissões.

### 4. Baixar e salvar a chave pública

```bash
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null
```

Essa linha combina quatro operações:

1. O `curl` baixa o arquivo da chave pública.
2. O símbolo `|`, chamado *pipe*, encaminha a saída do primeiro comando para a entrada do próximo.
3. O `sudo tee` grava o conteúdo recebido no arquivo indicado, com privilégios administrativos.
4. O `> /dev/null` descarta a cópia desse conteúdo que o `tee` normalmente exibiria no terminal.

As opções do `curl` significam:

| Opção | Função |
| --- | --- |
| `-f` | Faz o comando indicar falha diante de erros HTTP, como um arquivo não encontrado. |
| `-s` | Oculta o medidor de progresso e as mensagens usuais. |
| `-S` | Mantém a exibição de mensagens de erro em conjunto com `-s`. |
| `-L` | Segue redirecionamentos da URL. |

O `tee` é executado com `sudo` porque o arquivo será escrito em um diretório do sistema.

A chave pública permite ao APT verificar assinaturas da fonte de pacotes. Ela é diferente de uma credencial de acesso à sua conta do GitHub.

### 5. Permitir a leitura da chave

```bash
sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg
```

O `chmod` altera permissões de arquivos e diretórios:

- `g`: grupo.
- `o`: outros usuários.
- `+r`: acrescenta permissão de leitura.

Isso permite que os processos do APT que precisam ler a chave consigam acessá-la.

### 6. Cadastrar a fonte de pacotes

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
```

Essa linha informa ao APT onde encontrar os pacotes do GitHub CLI.

Aqui, “repositório de pacotes” significa uma fonte de instalação de software. É um uso diferente da palavra “repositório” quando falamos do código de um projeto.

| Trecho | Função |
| --- | --- |
| `echo` | Produz o texto da configuração. |
| `deb` | Identifica uma fonte de pacotes binários no formato Debian. |
| `$(dpkg --print-architecture)` | Executa o comando entre parênteses e insere a arquitetura do sistema, como `amd64` ou `arm64`. |
| `arch=...` | Limita a busca à arquitetura informada. |
| `signed-by=...` | Indica a chave que o APT deve usar para verificar essa fonte. |
| `https://cli.github.com/packages` | Informa o endereço da fonte de pacotes. |
| `stable main` | Identifica a distribuição e o componente definidos por essa fonte. |
| `sudo tee /etc/apt/sources.list.d/github-cli.list` | Salva a configuração no arquivo indicado. |
| `> /dev/null` | Descarta a cópia que o `tee` exibiria na tela. |

As aspas duplas mantêm o texto como um único argumento para o `echo` e permitem que `$(...)` seja substituído pelo resultado do comando.

### 7. Atualizar a lista novamente

```bash
sudo apt update
```

Como uma nova fonte foi adicionada, o APT precisa consultá-la para incluir seus pacotes no catálogo local.

### 8. Instalar o gh

```bash
sudo apt install gh
```

Agora o APT pode instalar o pacote `gh` a partir das fontes configuradas. Como não usamos `-y`, ele poderá solicitar uma confirmação antes de continuar.

Ao terminar, confira:

```bash
gh --version
```

Se o terminal mostrar a versão, o comando está disponível nesse ambiente.

## Autenticação no GitHub

Instalar a ferramenta e autenticar a conta são etapas diferentes.

Para iniciar:

```bash
gh auth login
```

Para acompanhar este exemplo, selecione, conforme as perguntas apresentadas:

1. **GitHub.com** como plataforma.
2. **HTTPS** como protocolo para operações Git.
3. A opção de entrar pelo navegador.

Se houver uma pergunta sobre autenticar o Git com suas credenciais do GitHub, responda “sim” caso queira configurar também esse acesso por HTTPS.

No fluxo pelo navegador, o terminal apresenta um código temporário e orienta você a abrir uma página do GitHub. Digite o código na página indicada e autorize o acesso.

Depois, volte ao terminal e confira:

```bash
gh auth status
```

Esse comando mostra o estado da autenticação.

Existe uma passagem pelo navegador nessa configuração inicial. O objetivo é reduzir as trocas de tela nas tarefas seguintes.

**Já estar conectada ao GitHub no navegador ou no VS Code não garante que o `gh` esteja autenticado no terminal utilizado.**

## Primeiros comandos no projeto

No terminal, entre na pasta do seu projeto:

```bash
cd caminho/do/seu/projeto
```

Substitua o caminho pelo local real. Os exemplos abaixo consideram um repositório local já vinculado a um repositório no GitHub.

### Consultar o repositório

```bash
gh repo view
```

Mostra informações do repositório identificado a partir do projeto atual.

### Alterar a descrição no GitHub

```bash
gh repo edit --description "Projetos de estudo em SQL e engenharia de dados"
```

| Trecho | Função |
| --- | --- |
| `gh` | Chama o GitHub CLI. |
| `repo` | Indica operações relacionadas a repositórios. |
| `edit` | Solicita uma alteração nas configurações. |
| `--description` | Informa qual configuração será alterada. |
| Texto entre aspas | Define a nova descrição. |

Essa alteração afeta a descrição exibida no GitHub. O conteúdo do README é editado separadamente.

### Consultar pull requests

```bash
gh pr list
```

O `pr` é a abreviação de *pull request*: uma proposta para incorporar alterações de uma branch em outra.

## Exemplo: atualizar o README e abrir um pull request

Imagine que quero atualizar o README e propor essa mudança por um pull request.

Partindo de uma branch base atualizada, crio uma branch para a alteração:

```bash
git switch -c docs/atualiza-readme
```

Depois de editar e salvar o README:

```bash
git status
git diff
git add README.md
git commit -m "docs: atualiza documentação do projeto"
git push -u origin docs/atualiza-readme
```

Cada linha tem uma função:

| Comando | Função |
| --- | --- |
| `git status` | Mostra a situação dos arquivos e da branch. |
| `git diff` | Permite revisar alterações ainda não preparadas para commit em arquivos já rastreados. |
| `git add README.md` | Prepara as alterações desse arquivo para o próximo commit. |
| `git commit -m "..."` | Registra localmente as alterações preparadas, com uma mensagem. |
| `git push -u origin docs/atualiza-readme` | Envia a branch ao remoto `origin` e configura seu vínculo de acompanhamento. |

O exemplo considera o README na raiz do repositório. Para um arquivo dentro de uma pasta, ajuste o caminho no `git add`, por exemplo: `git add vscode/README.md`.

Com a branch enviada, abro o pull request:

```bash
gh pr create
```

A ferramenta conduz o preenchimento das informações, como título, descrição e branch de destino. Ao concluir a criação, apresenta o link do pull request.

A criação do PR propõe a integração das mudanças; o merge é uma etapa posterior.

## Como consultar ajuda

Para explorar os comandos disponíveis:

```bash
gh --help
```

Para consultar um grupo específico:

```bash
gh repo --help
```

Para entender as opções de uma operação:

```bash
gh repo edit --help
```

Esse padrão permite aprender aos poucos, conforme surge uma necessidade no projeto.

## Sobre este tutorial

Este material nasceu de uma necessidade durante meus estudos e registra o caminho que usei para começar a trabalhar com o GitHub CLI pelo terminal integrado do VS Code.

Além de instalar a ferramenta, entender cada comando ajudou a compreender como o Linux instala programas, como o APT verifica uma fonte de pacotes e como diferentes comandos se conectam pelo terminal.

## Referências oficiais

- [Manual do GitHub CLI](https://cli.github.com/manual/)
- [Instalação no Linux](https://github.com/cli/cli/blob/trunk/docs/install_linux.md)
- [Autenticação: gh auth login](https://cli.github.com/manual/gh_auth_login)
- [Configurações: gh repo edit](https://cli.github.com/manual/gh_repo_edit)
- [Pull requests: gh pr create](https://cli.github.com/manual/gh_pr_create)
- [Manual do curl](https://curl.se/docs/manpage.html)