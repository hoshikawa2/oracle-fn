# OCI DEV Functions

O objetivo deste workshop é permitir ao participante que deseja conhecer um pouco do mundo de Serverless em cloud com functions.

Em nosso dia-a-dia, ouvimos muito sobre estes termos mas muitas vezes, entender cada conceito se torna algo bastante complexo. Certo?

O workshop irá compreender os seguintes conceitos:

- Cloud
- docker
- Linha de Comando OCI (OCI CLI)
- Linha de Comando fn (fn CLI)
- Compartimentos
- Usuários/Grupos/Políticas
- Virtual Cloud Network (VCN)
- Repositório de Imagens Docker (OCIR)
- Serverless (Oracle Cloud functions)
- Perfil para OCI CLI
- Deployments
- API Gateway

Esperamos que, ao final deste workshop, o participante possa entender esses conceitos e praticar cada vez mais neste mundo de aplicações distribuídas.


Este material foi baseado em materiais já existentes **Oracle** (vide seção **Referências** ao final) e foi organizado especialmente para tornar a experiência prática deste workshop mais agradável. O material original se encontra em inglês.

Observação: Este material faz referência às imagens ilustrativas originais, portanto, se a imagem não aparecer neste material, provavelmente é porque o material original foi reformulado por algum motivo. Pode ser que este motivo seja uma mudança de versão do tutorial ou mesmo do recurso cloud (functions, OCIR, OCI CLI, etc). Agradeço imensamente se puder avisar, pois poderemos atualizar este material com mais velocidade.

-----
# T1 - Tutorial 1 - Pré Check-in


### T1.1 - Criando uma conta na OCI

Abra a sua conta trial na Oracle Cloud Infrastructure.

Siga este link:

    https://videohub.oracle.com/media/1_m4904ocr
    

### T1.2 Instalando o Docker

O docker é uma alternativa à virtualização, onde o maior diferencial é o compartilhamento do Kernel da máquina hospedeira (Linux) com os containers que fazem o papel do ambiente virtualizado.
As aplicações assim podem se beneficiar da implantação nestes containers e contar com flexibilidade e portabilidade independente do ambiente em que sejam implatados (nuvem pública, nuvem privada, on-premisses, etc).
A instalação do docker é necessária para este workshop para execução de comandos de manipulação do repositório de imagens.

A instalação do docker segue o site oficial Docker, conforme abaixo:

    https://docs.docker.com/engine/install/centos/
    https://docs.docker.com/engine/install/debian/
    https://docs.docker.com/engine/install/ubuntu/
    https://docs.docker.com/docker-for-mac/install/
    https://docs.docker.com/docker-for-windows/install/
    

Se você pretende instalar o docker com WSL no Windows:

    WSL deve funcionar bem se você instalar o docker com WSL2 e Ubuntu
    Até o fechamento deste material, o WSL1 e Debian não funcionaram corretamente
    
**T1.2.1 Linux**

Para instalar o docker no Oracle Linux (utilize a versão 8), CentOS, Redhat, primeiramente faça o setup do repositório:

    $ sudo yum install -y yum-utils

    $ sudo yum-config-manager --add-repo https://download.docker.com/linux/<edição do Linux>/docker-ce.repo
    
    Edições de Linux disponíveis:
    centos (utilizar este para Oracle Linux 8)
    debian
    fedora
    raspbian
    rhel
    static
    ubuntu
    
    exemplo para centos:
    $ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

Em seguida, instale a última versão do Docker:

    $ sudo yum install docker-ce docker-ce-cli containerd.io

Será apresentado o fingerprint conforme abaixo. Assim que for apresentado, clique em aceitar.

    060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35

Execute o comando abaixo para poder trabalhar com o functions integrado ao docker:

    sudo chown opc:docker /var/run/docker.sock
    
    opc é o usuário padrão do Oracle Linux 8
    Se estiver instalando docker para Ubuntu, por exemplo, troque pelo usuário padrão de trabalho. Neste exemplo, utilize o usuário ubuntu

Inicie o docker com:

    $ sudo systemctl start docker
    
Teste para saber se o docker está funcionando:

    $ sudo docker run hello-world
    

**T1.2.2 Ubuntu**

Para instalar o docker no Ubuntu, primeiramente faça o setup do repositório:

    $ sudo apt-get update

    $ sudo apt-get install \
      apt-transport-https \
      ca-certificates \
      curl \
      gnupg-agent \
      software-properties-common
    
Adicione o GPG key oficial do docker:

    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

    $ sudo apt-key fingerprint 0EBFCD88

Atualize o repositório:

    $ sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \      
      $(lsb_release -cs) \
      stable"
    $ sudo apt-get update
          
Em seguida, instale a última versão do Docker:

    $ sudo apt-get install docker-ce docker-ce-cli containerd.io
    $ sudo /etc/init.d/docker start
       
Teste para saber se o docker está funcionando:

    $ sudo docker run hello-world
    
**T1.2.3 Mac OS X**

Faça o download na página do docker para Mac OS X:

    https://hub.docker.com/editions/community/docker-ce-desktop-mac/
    
Dê duplo-clique com o mouse no arquivo Docker.dmg e siga os passos.

Na janela do terminal, teste o docker com o comando:

    $ sudo docker run hello-world

**T1.2.4 Windows**

Faça o download na página do docker para Mac OS X:

    https://hub.docker.com/editions/community/docker-ce-desktop-windows/
    
Dê duplo-clique com o mouse no arquivo "Docker for Windows Installer" e siga os passos.

Na janela do Powershell, teste o docker com o comando:

    $ docker run hello-world
    
----
# T2 - Tutorial 2 - Instalando o CLI

O OCI CLI (Command Line Interface) permite trabalhar com os recursos da Cloud Oracle através de linhas de comando.
Além disto, para trabalhar com a linha de comandos para Kubernetes (kubectl), é necessário integrar o kubectl com o OCI CLI.
Neste tutorial iremos instalar o CLI como prévia para que você possa então, nos próximos passos, configurar seu kubectl para integrar-se à OCI.
Vamos lá.

    O script do instalador instala automaticamente o CLI e suas dependências, Python e virtualenv. 

**Antes de executar o instalador, certifique-se de atender aos requisitos:**

Versões e sistemas operacionais compatíveis com Python

- A CLI é compatível com Python versões 2.7+ e 3.5 a 3.8 em execução em MacOS, Windows ou uma distribuição Linux compatível:

- Oracle Linux 6.10, Oracle Linux 7.7 e 7.8 e Oracle Linux 8.0
- Oracle Autonomous Linux 7.8
- CentOS 6.9, CentOS 6.10 e CentOS 7.0
- Ubuntu 16.04, Ubuntu 18.04 e Ubuntu 20.04

### T2.1 - Linux e Unix (incluindo Oracle Linux 8, Ubuntu)

**T2.1.1** Abra um terminal.

**T2.1.2** Para executar o script do instalador, execute o seguinte comando.

    bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"

**Nota**

    Para executar uma instalação 'silenciosa' que aceita todos os valores padrão sem prompts, use o parâmetro --accept-all-defaults

**T2.1.3** Responda às solicitações do script de instalação .

### T2.2 - Oracle Linux 7

Se você estiver usando o Oracle Linux 7, você pode usar o yum para instalar o CLI.

**T2.2.1** Para usar o yum para instalar a CLI:

    sudo yum install python36-oci-cli

A CLI será instalada nos pacotes do site Python:

    /usr/lib/python3.6/site-packages/oci_cli
    /usr/lib/python3.6/site-packages/services

A documentação e os exemplos serão instalados no /usr/share/doc/python36-oci-cli-<version>/diretório.

**T2.2.2** Para desinstalar a CLI:

    sudo yum remove python36-oci-cli

### T2.3 - Mac OS X

Você pode usar o Homebrew para instalar, atualizar e desinstalar o CLI no Mac OS.

**T2.3.1** Para instalar a CLI no Mac OS X com Homebrew:

    brew update && brew install oci-cli

**T2.3.2** Para atualizar sua CLI, instale no Mac OS X usando o Homebrew:

    brew update && brew upgrade oci-cli

**T2.3.3** Para desinstalar a CLI no Mac OS X usando o Homebrew:

    brew uninstall oci-cli

### T2.4 - Windows

**T2.4.1** Abra o console do PowerShell usando a opção Executar **como administrador** .

O instalador permite o preenchimento automático instalando e executando um script. Para permitir que este script seja executado, você deve habilitar a política de execução **RemoteSigned**.

Para configurar a política de execução remota para PowerShell, execute o seguinte comando.

    Set-ExecutionPolicy RemoteSigned

**T2.4.2** Baixe o script do instalador:

    Invoke-WebRequest https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.ps1 -OutFile install.ps1

**T2.4.3** Execute o script do instalador com ou sem prompts:

**T2.4.3.1** Para executar o script do instalador com prompts, execute o seguinte comando:

    iex ((New-ObjectSystem.Net.WebClient) .DownloadString ('https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.ps1'))

... e responda às solicitações do script de instalação .


**T2.4.4** Para executar o script do instalador sem avisar o usuário, aceitando as configurações padrão, execute o seguinte comando:


       install.ps1 -AcceptAllDefaults 


### T2.5 - Instruções do script de instalação

O script de instalação solicita as seguintes informações.

    Se você não tiver uma versão compatível do Python instalada:

        Windows e Linux: Você é solicitado a fornecer um local para instalar os binários
        e executáveis. O script instalará o Python para você.

        MacOS: você é notificado de que sua versão do Python é incompatível. Você deve
        atualizar antes de prosseguir com a instalação. O script não instalará o 
        Python para você.

    Quando solicitado a atualizar o CLI para a versão mais recente, responda 
    com Y para sobrescrever uma instalação existente.
    Quando solicitado a atualizar seu PATH, responda com Y para poder invocar a 
    CLI sem fornecer o caminho completo para o executável. Isso adicionará oci.exe 
    ao seu PATH.

### T2.6 - Configurando o arquivo config

Antes de usar a CLI, você deve criar um arquivo de configuração que contém as credenciais necessárias para trabalhar com o **Oracle Cloud Infrastructure** .

Para isto, vamos seguir o roteiro:
- Buscar as informações de Tenant e Usuário (OCID)
- Subir a chave pública para o seu usuário
- Executar o comando para configurar o OCI CLI

Vamos lá

 
**T2.6.1 - OCI do Tenant**

**T2.6.1.1** Clique em Configurações do usuário conforme a imagem e em seguida clique no Tenant:

![Configuração do Tenant](https://github.com/hoshikawa2/OCI-DEV/blob/main/images/Acessando%20Tenant%201.png?raw=true)

**T2.6.1.2** O OCID do tenant é mostrado em Informações de tenant . 

![OCID do Tenant](https://github.com/hoshikawa2/OCI-DEV/blob/main/images/Acessando%20Tenant%202.png?raw=true)


**T2.6.1.3** Clique em Copiar para copiá-lo para a área de transferência e guarde em suas anotações para uso posterior


**T2.6.2 - OCI do Usuário**

**T2.6.2.1** Se você estiver conectado como o usuário:

**T2.6.2.1.1** Clique em Configurações do usuário conforme a imagem e em seguida clique no seu usuário:

![Configuração do Usuário](https://github.com/hoshikawa2/OCI-DEV/blob/main/images/Acessando%20Usuario%201.png?raw=true)

**T2.6.2.1.2** Se você for um administrador fazendo isso para outro usuário: Abra o menu de navegação . Em Governança e Administração , acesse Identidade e clique em Usuários . Selecione o usuário na lista.

**T2.6.2.1.3** O OCID do usuário é mostrado em Informações do usuário . 

![OCID do Usuário](https://github.com/hoshikawa2/OCI-DEV/blob/main/images/Acessando%20Usuario%202.png?raw=true)

**T2.6.2.1.4** Clique em Copiar para copiá-lo para a área de transferência e guarde em suas anotações para uso posterior


**T2.6.3 Configurando OCI CLI**

Para que a CLI o guie pelo processo de configuração inicial, use o comando:

    oci setup config


**O comando solicita as informações necessárias para o arquivo de configuração e as chaves públicas / privadas da API.**

    Obs: Ao executar o comando oci setup config, você será questionado se deseja carregar
    suas chaves previamente geradas ou se deseja gerar as chaves. Informe que deseja gerar as chaves.
    Lembre que esta etapa pede para você determinar o diretório em que as chaves estão
    ou que você deseja ter como padrão. Normalmente .oci

Você vai precisar de algumas informações para preencher a configuração quando executar o comando **oci setup config**. Estas informações você anotou nos passos anteriores.

- OCID do Tenant
- OCID do seu Usuário na Cloud Oracle


**T2.6.4 - Como fazer upload da chave pública**

Você pode fazer upload da chave pública PEM no console , que pode ser acessado fazendo login aqui: https://cloud.oracle.com

**T2.6.4.1** Clique em Configurações do usuário conforme a imagem e em seguida clique no seu usuário:

![Configuração do Usuário](https://github.com/hoshikawa2/OCI-DEV/blob/main/images/Acessando%20Usuario%201.png?raw=true)

**T2.6.4.2** Visualize os detalhes do usuário que chamará a API com o par de chaves:

**T2.6.4.3** Se você for um administrador fazendo isso para outro usuário: 
- Abra o menu de navegação . 
- Em Governança e Administração , acesse Identidade e clique em Usuários . 
- Selecione o usuário na lista.

**T2.6.4.4** Clique em Adicionar chave pública .

![Adicionando chave pública](https://github.com/hoshikawa2/OCI-DEV/blob/main/images/API%20Key.png?raw=true)

**T2.6.4.5** Cole o conteúdo da chave pública PEM na caixa de diálogo e clique em Adicionar .

    A impressão digital da chave é exibida (por exemplo, 12: 34: 56: 78: 90: ab: cd: ef: 12: 34: 56: 78: 90: ab: cd: ef).


**T2.6.5 - Testando o OCI CLI**

Para testar suas configurações, execute o seguinte comando:

    $ oci iam user list

Você deve obter algo como:

	{
	  "data": [
	    {
	      "capabilities": {
	        "can-use-api-keys": true,
	        "can-use-auth-tokens": true,
	        "can-use-console-password": true,
	        "can-use-customer-secret-keys": true,
	        "can-use-o-auth2-client-credentials": true,
	        "can-use-smtp-credentials": true
	      },
	      "compartment-id": "ocid1.tenancy.oc1..aaaaaaaarmfxxxxxxxxxxxxyazqycjapqsyaxxxxxxxxxxxxxxbyb6j6q",
	      "defined-tags": {},
	      "description": "Cristiano Hoshikawa",
	      "email": "teste@teste.com",
	      "email-verified": true,
	      "external-identifier": null,
	      "freeform-tags": {},
	      "id": "ocid1.user.oc1..aaaaaaxccccccccccccdpmntisfgxeizqwwwwwewwweeeesol4f3keeeeeexmlcmphda",
	      "identity-provider-id": null,
	      "inactive-status": null,
	      "is-mfa-activated": false,
	      "lifecycle-state": "ACTIVE",
	      "name": "teste@teste.com",
	      "time-created": "2020-04-17T10:57:09.254000+00:00"
	    }
	}


----


# T3 - Tutorial 3 - Instalando functions CLI


Faça login em seu ambiente de desenvolvimento como desenvolvedor de funções e:

Abra o arquivo Fn Project CLI README.md no GitHub e siga as instruções apropriadas para instalar o Fn Project CLI. Como uma visão geral conveniente, as instruções são resumidas abaixo:

    Linux ou MacOS: Digite:
    $ curl -LSs https://raw.githubusercontent.com/fnproject/cli/master/install | sh
    
    MacOS usando Homebrew: Digite:
    $ brew update && brew install fn
    
    Linux, MacOS ou Windows: 
    Baixe e execute o binário da página Fn Project Releases no GitHub
    https://github.com/fnproject/cli/releases

Confirme se a CLI foi instalada inserindo:

    $ fn version

----

# T4 - Tutorial 4 - Usando o OCIR (Oracle Cloud Infrastructure Registry)


Este tutorial de 10 minutos mostra como:

- criar um token de autenticação para usar com o Oracle Cloud Infrastructure Registry
- faça login no Oracle Cloud Infrastructure Registry a partir do Docker CLI
- extrair uma imagem de teste do DockerHub
- colocando uma tag em uma imagem
- envie a imagem para o Oracle Cloud Infrastructure Registry usando Docker CLI
- verificar se a imagem foi enviada para o Oracle Cloud Infrastructure Registry usando o console
fundo

### T4.1 - Introdução

O Oracle Cloud Infrastructure Registry é um registro gerenciado pela Oracle que permite simplificar o fluxo de trabalho de desenvolvimento para produção. O Oracle Cloud Infrastructure Registry torna mais fácil para você, como desenvolvedor, armazenar, compartilhar e gerenciar artefatos de desenvolvimento como imagens Docker. E a arquitetura altamente disponível e escalonável do Oracle Cloud Infrastructure garante que você possa implantar seus aplicativos de maneira confiável. Assim, você não precisa se preocupar com problemas operacionais ou com o dimensionamento da infraestrutura subjacente.

Neste tutorial, você primeiro criará um token de autenticação para acessar o Oracle Cloud Infrastructure Registry. Em seguida, você obterá uma imagem de teste do DockerHub e dará a ela uma nova tag. A nova tag identifica a região, tenant e repositório do Oracle Cloud Infrastructure Registry para o qual você deseja enviar a imagem.

Depois de fornecer a tag à imagem, você a envia por push para o Oracle Cloud Infrastructure Registry usando o Docker CLI. Por fim, você verificará se a imagem foi enviada com sucesso, visualizando o repositório que foi criado.

### T4.2 - O que você precisa?

Um nome de usuário e senha do Oracle Cloud Infrastructure.
Para enviar imagens para o Oracle Cloud Infrastructure Registry, você deve pertencer a um dos seguintes:
- O grupo Administradores da tenant.
- Um grupo ao qual uma política concede as permissões apropriadas do Oracle Cloud Infrastructure Registry, incluindo a permissão REPOSITORY_CREATE (consulte o tópico Políticas para controlar o acesso ao repositório na documentação do Oracle Cloud Infrastructure Registry).
- Acesso ao Docker CLI. Por exemplo, para enviar e receber imagens em uma máquina cliente local, conforme descrito neste tutorial, você precisará ter instalado o Docker na máquina local. (Como alternativa, você pode usar o Docker no ambiente do Cloud Shell.)

### T4.3 - Iniciar Oracle Cloud Infrastructure

**T4.3.1** Em um navegador, acesse o url que você recebeu para fazer login no Oracle Cloud Infrastructure.

![Página de login](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-login-page.png)

**T4.3.2** Especifique o seu tenant na qual você tenha as permissões apropriadas para criar repositórios no Oracle Cloud Infrastructure Registry. Você herda essas permissões de uma das seguintes maneiras:

- por pertencer ao grupo Administradores do locatário
- por pertencer a outro grupo ao qual uma política concede as permissões apropriadas do Oracle Cloud Infrastructure Registry, incluindo a permissão REPOSITORY_CREATE

Este tutorial assume que a tenant é acme-dev.

**T4.3.3** Digite seu nome de usuário e senha e clique em Entrar . 

Este tutorial assume que o nome de usuário é jdoe@acme.com.

### T4.4 - Obtenha um token de autenticação

**T4.4.1** No canto superior direito do console, abra o menu Usuário ( Menu do usuário) e clique em Configurações do usuário .

![Tela do console](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-console-settings.png)

**T4.4.2** Na página Tokens de autenticação , clique em Gerar token .

![Caixa de diálogo de geração de token](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-auth-tokens.png)

**T4.4.3** Insira Tutorial auth tokenuma descrição amigável para o token de autenticação e clique em Gerar token . O novo token de autenticação é exibido.

**T4.4.4** Copie o token de autenticação imediatamente para um local seguro de onde você possa recuperá-lo mais tarde, porque você não verá o token de autenticação novamente no console.

**T4.4.5** Feche a caixa de diálogo Gerar token.

Confirme se você pode acessar o Oracle Cloud Infrastructure Registry:
- No console, abra o menu de navegação. Em Soluções e plataforma , acesse Serviços para desenvolvedores e clique em Container Registry .
- Escolha a região em que você estará trabalhando (por exemplo, us-ashburn-1). 
- Revise os repositórios que já existem. Este tutorial assume que nenhum repositório foi criado ainda.

![Página de registro](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-registry-no-images.png)

### T4.5 - Faça login no Oracle Cloud Infrastructure Registry a partir do Docker CLI

**T4.5.1** Em uma janela de terminal na máquina cliente que executa o Docker, faça login no Oracle Cloud Infrastructure Registry digitando:

    docker login <region-key>.ocir.io
    
onde **<region-key>** está a chave para a região do Oracle Cloud Infrastructure Registry que você está usando. 

    Por exemplo phx,. Consulte o tópico Disponibilidade por região na documentação do Oracle Cloud Infrastructure Registry.
    DICA: As regiões são as siglas dos aeroportos. Por exemplo: GRU - Guarulhos-SP, IAD - Ashburn, etc

**T4.5.2** Quando solicitado, digite seu nome de usuário no formato < tenancy-namespace >/< username >. 

Por exemplo ansh81vru1zp/jdoe@acme.com. Se a sua tenant for federada com o Oracle Identity Cloud Service, use o formato < tenancy-namespace >/oracleidentitycloudservice/< username >.

**T4.5.3** Quando solicitado, digite o token de autenticação que você copiou anteriormente como a senha.

![Janela do terminal](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-docker-login.png)

### T4.6 - Extraia a imagem hello-world do DockerHub

Em uma janela de terminal na máquina cliente que executa o Docker
Digite 

    docker pull karthequian/helloworld:latest 
    
para recuperar a versão mais recente da imagem hello-world do DockerHub.

![Janela do terminal](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-docker-pull.png)

As diferentes camadas da imagem do helloworld são puxadas separadamente.

### T4.7 - Marque a imagem para push

Em uma janela de terminal na máquina cliente que executa o Docker, dê uma tag à imagem que você vai enviar para o Oracle Cloud Infrastructure Registry inserindo:

    docker tag karthequian/helloworld:latest
    <region-key>.ocir.io/<tenancy-namespace>/<repo-name>/<image-name>:<tag>
    
Onde:

    <region-key> é a chave para a região do Oracle Cloud Infrastructure Registry que você está usando. Por exemplo phx,. Consulte o tópico Disponibilidade por região na documentação do Oracle Cloud Infrastructure Registry.
        
    ocir.io é o nome do Oracle Cloud Infrastructure Registry.

    <tenancy-namespace>é a string de namespace de armazenamento de objeto gerada automaticamente da tenant (conforme mostrado na página Informações de tenant ) para a qual você deseja enviar a imagem. Por exemplo, o namespace da acme-devtenant pode ser ansh81vru1zp. Observe que seu usuário deve ter acesso ao arrendamento.

    <repo-name>(se especificado) é o nome de um repositório para o qual você deseja enviar a imagem (por exemplo, project01). Observe que especificar um repositório é opcional. Se você não especificar um nome de repositório, o nome da imagem será usado como o nome do repositório no Oracle Cloud Infrastructure Registry.

    <image-name>é o nome que você deseja dar à imagem no Oracle Cloud Infrastructure Registry (por exemplo, helloworld).

    <tag>é uma marca de imagem que você deseja fornecer à imagem no Oracle Cloud Infrastructure Registry (por exemplo, latest).
    
Por exemplo:

    docker tag karthequian/helloworld:latest phx.ocir.io/ansh81vru1zp/helloworld:latest
    
![Janela do terminal](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-docker-tag.png)

**T4.7.1** Revise a lista de imagens disponíveis digitando:

    docker images
    
![Janela do terminal](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-docker-images.png)

Observe que, embora duas imagens marcadas sejam mostradas, ambas são baseadas na mesma imagem (com o mesmo id de imagem).

### T4.8 - Envie a imagem hello-world para o Oracle Cloud Infrastructure Registry

Em uma janela de terminal na máquina cliente que executa o Docker, envie a imagem Docker da máquina cliente para o Oracle Cloud Infrastructure Registry inserindo:

    docker push <region-key>.ocir.io/<tenancy-namespace>/<repo-name>/<image-name>:<tag>
    
Onde:

    <region-key>é a chave para a região do Oracle Cloud Infrastructure Registry que você está usando. Por exemplo phx,. Consulte o tópico Disponibilidade por região na documentação do Oracle Cloud Infrastructure Registry.
ocir.io é o nome do Oracle Cloud Infrastructure Registry.

    <tenancy-namespace>é a string de namespace de armazenamento de objeto gerada automaticamente da tenant (conforme mostrado na página Informações de tenant ) que possui o repositório para o qual você deseja enviar a imagem. Por exemplo, o namespace da acme-devtenant pode ser ansh81vru1zp. Observe que seu usuário deve ter acesso ao arrendamento.

    <repo-name>(se especificado) é o nome de um repositório para o qual você deseja enviar a imagem (por exemplo, project01). Observe que especificar um repositório é opcional. Se você não especificar um nome de repositório, o nome da imagem será usado como o nome do repositório no Oracle Cloud Infrastructure Registry.

    <image-name>é o nome que você deseja dar à imagem no Oracle Cloud Infrastructure Registry (por exemplo, helloworld).

    <tag>é uma marca de imagem que você deseja fornecer à imagem no Oracle Cloud Infrastructure Registry (por exemplo, latest).

Por exemplo:

    docker push phx.ocir.io/ansh81vru1zp/helloworld:latest

![Janela do terminal](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-docker-push.png)

As diferentes camadas da imagem do helloworld são empurradas uma por uma.

### T4.9 - Verifique se a imagem foi enviada ao Oracle Cloud Infrastructure Registry

**T4.9.1** Na janela do navegador que mostra o Console com a página Registro exibida, clique em Recarregar . Você vê todos os repositórios no registro aos quais tem acesso, incluindo o repositório privado helloworld que foi criado quando você enviou a imagem helloworld.

![Página de registro](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-registry-repositories.png)

**T4.9.2** Clique no nome do repositório helloworld que contém a imagem que você acabou de enviar. 

Veja que:

- As diferentes imagens no repositório. Nesse caso, há apenas uma imagem, com a tag mais recente.
- Detalhes sobre o repositório, incluindo quem o criou e quando, seu tamanho e se é um repositório público ou privado
- O Readme associado ao repositório. Nesse caso, ainda não há Readme.

![Página de registro](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-repository-images.png)

**T4.9.3** Forneça um leia-me para o repositório helloworld da seguinte maneira:

**T4.9.3.1** Clique no botão Editar na seção Readme .

**T4.9.3.2** Em Edit Readme, selecione a opção Markdown e copie e cole a seguinte descrição da imagem helloworld no conteúdo de campo:

    ## Exemplo Hello World	
    por Karthequian [retirado do Dockerhub](https://hub.docker.com/r/karthequian/helloworld/)
    
    ![Hellow World por Karthequian](https://github.com/hoshikawa2/repo-image/blob/master/imagem_hello_world.png?raw=true)


![Janela Editar Leiame, guia Editar](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-image-readme-complete.png)

**T4.9.3.3** Clique na guia Visualizar para ver como o leiame aparecerá.

![Janela Editar Leiame, guia Visualizar](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-image-readme-preview.png)

**T4.9.3.4** Clique em Salvar para fechar a caixa de diálogo Editar Leiame.

**T4.9.4** Clique na última marca de imagem. A seção Detalhes mostra o tamanho da imagem, quando ela foi empurrada e por qual usuário, e o número de vezes que a imagem foi puxada.

![Página de registro](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-image-summary.png)

**(Opcional)** Mais tarde, se você deseja puxar a imagem, clique no botão Ações ao lado do nome da imagem e selecione Copiar comando de puxar . Por exemplo, o comando pode ser:
**docker pull phx.ocir.io/ansh81vru1zp/helloworld:latest**

Parabéns! Você puxou com sucesso a imagem helloworld do DockerHub, marcou-a e enviou-a para o Oracle Cloud Infrastructure Registry usando o Docker CLI. Você verificou que a imagem foi enviada com sucesso e adicionou uma descrição no readme.

----

# T5 - Criando seu primeiro projeto functions

Esta é a etapa onde criaremos as functions. Após instalar e configurar toda a estrutura para poder trabalhar com Oracle functions, neste ponto iremos configurar sua primeira função e testá-la.

**T5.1 Configure sua locação**

Este passo é muito importante, pois criaremos aqui o usuário a trabalhar com o Oracle functions.
Também criaremos o grupo e compartimento para armazenar a aplicação e a função deste workshop, bem como daremos os acessos para este usuário.
É muito importante que este usuário, não-federado, seja criado para se trabalhar com o Oracle functions, pois não é possível trabalhar com o usuário federado no exemplo a ser seguido.
Distingue-se o usuário federado do não-federado de forma muito simples. Usuários federados iniciam seu nome com **oracleidentitycloudservice/** à frente do nome do usuário.

    Você poderá criar seu usuário não-federado com o mesmo nome do usuário federado.
    Logo, se seu usuário federado é oracleidentitycloudservice/joao.almeida@oracle.com
    Seu usuário não federado será: joao.almeida@oracle.com


**T5.1.1 Crie grupos e usuários**
	
Como prática recomendada, você deve criar grupos com base nas atribuições da sua organização, que geralmente se enquadram nas atribuições típicas da organização. Em seguida, designe as atribuições do usuário apropriadas a esses grupos para dar-lhes acesso às funcionalidades necessárias. Finalmente, adicione usuários a esses grupos para designar automaticamente a eles as atribuições de usuário apropriadas.
		
Se os usuários e grupos adequados ainda não existirem, faça login no console como administrador de locação e, em Governança e administração , acesse Identidade e:
	
Crie um novo grupo clicando em Grupos e depois em Criar Grupo .

![create_group.png](https://github.com/hoshikawa2/repo-image/blob/master/create_group.png?raw=true)


Crie um novo usuário clicando em Usuários e em Criar usuário. Lembre-se de selecionar "IAM User". Após criar o usuário, este usuário irá receber um e-mail para ajustar sua senha.

![criar_usuario2.png](https://github.com/hoshikawa2/repo-image/blob/master/criar_usuario2.png?raw=true)

Adicione um usuário a um grupo clicando em Grupos , depois no nome do grupo e em Adicionar usuário ao grupo .

![adicionar_usuario_grupo.png](https://github.com/hoshikawa2/repo-image/blob/master/adicionar_usuario_grupo.png?raw=true)



**T5.1.2 Criar compartimento**
	
Compartimentos no Oracle Cloud Infrastructure Identity and Access Management (IAM) são estruturas lógicas que ajudam a organizar e controlar o acesso aos recursos de nuvem.
	
Um compartimento do IAM contém recursos, como instâncias de banco de dados, redes virtuais na nuvem e volumes em blocos. Pense em um compartimento como um grupo lógico e não um contêiner físico. Ele age como um filtro para o que você está exibindo. Sempre que você adicionar um recurso no Oracle Cloud Infrastructure, crie-o em um compartimento específico. Se necessário, você pode mover recursos de um compartimento para outro. Os usuários precisam de permissões para acessar compartimentos e recursos neles.
	
**T5.1.2.1 Criar página de compartimento**

Se ainda não houver um compartimento adequado para criar recursos de rede e recursos de funções Oracle, faça login no console como administrador de locação e, em Governança e administração , acesse Identidade e:

Clique em Compartments e em Create Compartment.

![Criar_compartment.png](https://github.com/hoshikawa2/repo-image/blob/master/Criar_compartment.png?raw=true)


**T5.1.3 Crie VCN e sub-redes**
	
Quando você trabalha com o Oracle Cloud Infrastructure, uma das primeiras etapas é configurar uma rede virtual na nuvem (VCN) para seus recursos de nuvem.
Uma rede virtual privada configurada nos data centers da Oracle. Ela se parece muito com uma rede tradicional, com regras de firewall e tipos específicos de gateways de comunicação que você pode optar por usar. Uma VCN reside em uma única região do Oracle Cloud Infrastructure e abrange um único bloco CIDR IPv4 contíguo da sua escolha.
	
Sub-redes são subdivisões definidas em uma VCN (por exemplo, 10.0.0.0/24 e 10.0.1.0/24). As sub-redes contêm VNICs (virtual network interface cards), que são anexadas às instâncias. Cada sub-rede consiste em um intervalo contíguo de endereços IP que não se sobrepõem com outras sub-redes da VCN.
	
**T5.1.3.1 Iniciar caixa de diálogo do assistente VCN**

Se ainda não houver um VCN adequado para criar recursos de rede, faça login no console como administrador de locação e em Infraestrutura central , vá para Rede e, em seguida:

* Clique em Redes e em VCN (Virtual Cloud Network) e escolha um compartimento.
* Clique em Start VCN Wizard , depois em VCN with Internet Connectivity e em Start VCN Wizard .
* Insira um nome para o novo VCN, clique em Avançar e em Criar para criar o VCN junto com os recursos de rede relacionados.


![start_vcn_wizard.png](https://github.com/hoshikawa2/repo-image/blob/master/start_vcn_wizard.png?raw=true)

![vcn_wizard.png](https://github.com/hoshikawa2/repo-image/blob/master/vcn_wizard.png?raw=true)

![dados_vcn2.png](https://github.com/hoshikawa2/repo-image/blob/master/dados_vcn2.png?raw=true)


**T5.1.4 Criar política para o grupo**
	
Policy é um documento que especifica quem pode acessar quais recursos e como. O acesso é concedido no nível do grupo e do compartimento, o que significa que você pode escrever uma política que forneça a um grupo um tipo específico de acesso dentro de um compartimento específico ou ao próprio aluguel. Se você conceder a um grupo acesso à locação, o grupo obterá automaticamente o mesmo tipo de acesso a todos os compartimentos dentro da locação
	
* Faça login no console como administrador de locação e, em Governança e administração , acesse Identidade e clique em Políticas e, em seguida:

![criar_politicas.png](https://github.com/hoshikawa2/repo-image/blob/master/criar_politicas.png?raw=true)

![dados_politica.png](https://github.com/hoshikawa2/repo-image/blob/master/dados_politica.png?raw=true)

Clique na opção "Show manual editor"

* Se um ou mais usuários do Oracle Functions não for um administrador de locação, use o Criador de políticas para criar uma nova política. Selecione o compartimento raiz da locação e baseie a política no modelo de política. Permitir que os usuários criem, implantem e gerenciem funções e aplicativos usando o Cloud Shell.

O modelo de política inclui as seguintes declarações de política:


    Allow group <group-name> to use cloud-shell in tenancy
    Allow group <group-name> to manage repos in tenancy
    Allow group <group-name> to read objectstorage-namespaces in tenancy
    Allow group <group-name> to read metrics in tenancy
    Allow group <group-name> to manage functions-family in tenancy
    Allow group <group-name> to use virtual-network-family in tenancy
    
* Se necessário, você pode restringir essas declarações de política por compartimento (conforme descrito na documentação ).

**T5.2 Configure seu ambiente de desenvolvimento**

Aqui vamos iniciar a configuração do fn CLI em sua máquina local para poder implementar e implantar suas funções.
Algumas etapas são necessárias e iremos descrevê-las no decorrer deste tópico.
	
**T5.2.2.1 Configure Fn Project CLI no ambiente de desenvolvimento local**

**T5.5.2.1.1 Configurar chave de assinatura**

	
Nesta etapa, vamos configurar uma chave pública baseada na chave já criada anteriormente quando você instalou o OCI CLI.
Esta chave pública será utilizada pelo seu usuário não-federado que você acabou de criar para poder trabalhar com o Oracle Functions.
	
* Faça login em seu ambiente de desenvolvimento de máquina local como desenvolvedor de funções e:

Gere uma chave pública (criptografada com a mesma senha longa que você forneceu ao criar a chave privada e no mesmo local do arquivo da chave privada) inserindo:

    $ openssl rsa -pubout -in ~/.oci/<private-key-file-name>.pem -out ~/.oci/<public-key-file-name>.pem

Obs: Se você aceitou todos os defaults na etapa de configuração do OCI CLI, o parâmetro **private-key-file-name** é **oci_api_key.pem**

Copie o conteúdo do arquivo de chave pública que você acabou de criar, digitando:

    $ cat ~/.oci/<public-key-file-name>.pem | pbcopy

Criar chave API
![chave_api2.png](https://github.com/hoshikawa2/repo-image/blob/master/chave_api2.png?raw=true)

Faça login no console como desenvolvedor de funções, abra o menu do usuário ( Ícone do menu do usuário) e vá para as configurações do usuário . Na página Chaves de API , clique em Adicionar chave pública . Cole o valor da chave pública na janela e clique em Adicionar . A chave é carregada e sua impressão digital é exibida.



**T5.2.2.1.2 Configurar perfil OCI**
	
Antes de usar o Oracle Functions, você deve ter um arquivo de configuração CLI do Oracle Cloud Infrastructure que contém as credenciais da conta do usuário que você usará para criar e implantar funções. Essas credenciais de conta de usuário são chamadas de 'perfil'.
	
Iremos configurar o perfil para o OCI CLI para seu usuário (não-federado) criado no passo anterior **T5.1.1**

    Obs: Quando você criou o ambiente TRIAL, seu usuário foi criado como federado (o nome do seu usuário inicia-se por oracleidentitycloudservice/<nome do usuário>. Porém para trabalhar com o functions, estaremos utilizando o usuário não-federado (sem o oracleidentitycloudservice) pois o Grupo criado anteriormente não aceita usuários federados.

Faça login em seu ambiente de desenvolvimento de máquina local como desenvolvedor de funções e:

Abra o arquivo **~/.oci/config** em um editor de texto. (Se o diretório e / ou o arquivo ainda não existirem, crie-os).

Adicione um novo perfil ao arquivo ~ .oci / config da seguinte maneira:

    [<profile-name>]
    user=<user-ocid>
    fingerprint=<public-key-fingerprint>
    key_file=<full-path-to-private-key-pem-file>
    tenancy=<tenancy-ocid>
    region=<region-name>
    pass_phrase=<passphrase>

    <user-ocid> e <public-key-fingerprint> podem ser obtidos na tela do seu usuário (não-federado) recém criado
    <full-path-to-private-key-pem-file> é o diretório (se você aceitou todos os defaults na etapa de configuração do OCI CLI) ~/.oci/oci_api_key.pem
    <tenancy-ocid> é o mesmo indicado na configuração do seu OCI CLI para seu usuário federado

Salve e feche o arquivo.

**T5.2.2.1.3 Configure o contexto CLI do projeto Fn - provedor Oracle** 
	
Antes de usar o Oracle Functions , você deve configurar o Fn Project CLI para se conectar à sua locação do Oracle Cloud Infrastructure .
	
Quando o Fn Project CLI é inicialmente instalado, ele é configurado para um 'contexto' de desenvolvimento local. Para configurar o Fn Project CLI para se conectar à sua locação do Oracle Cloud Infrastructure , você deve criar um novo contexto. O contexto especifica os pontos de extremidade das Funções Oracle , o OCID do compartimento ao qual as funções implantadas pertencerão e o endereço do registro do Docker de e para o qual enviar e receber imagens.
	
Você pode definir vários contextos, cada um armazenado em um arquivo de contexto diferente no formato .yaml. Por padrão, os arquivos de contexto individuais são armazenados no diretório ~/.fn/contexts. O arquivo ~/.fn/config.yaml especifica qual arquivo de contexto o Fn Project usa.
	
Faça login em seu ambiente de desenvolvimento de máquina local como desenvolvedor de funções e:

Crie um novo contexto Fn CLI inserindo:

    $ fn create context meu_contexto --provider oracle

Observe que você especifica --provider oraclea ativação da autenticação e autorização usando assinatura de solicitação do Oracle Cloud Infrastructure, chaves privadas, grupos de usuários e políticas que concedem permissões a esses grupos de usuários.

Especifique que a CLI do projeto Fn deve usar o novo contexto inserindo:

    $ fn use context meu_contexto

Configure o novo contexto com o nome do perfil OCI que você criou para usar com o Oracle Functions, digitando:

    $ fn update context oracle.profile <profile-name>

Lembre-se: <profile-name> é o nome do perfil criado na etapa anterior. O nome do perfil é case-sensitive!!!!

**T5.2.2.1.4 Concluir a configuração do contexto CLI do projeto Fn**

Faça login em seu ambiente de desenvolvimento como desenvolvedor de funções e:

Configure o novo contexto com o OCID do compartimento em que deseja possuir as funções implantadas:

    $ fn update context oracle.compartment-id <compartment-ocid>

![compartment_ocid.png](https://github.com/hoshikawa2/repo-image/blob/master/compartment_ocid.png?raw=true)


Configure o novo contexto com o endpoint api-url para usar ao chamar a API OCI inserindo:

    $ fn update context api-url https://functions.<region>.oci.oraclecloud.com

    Onde <region> pode ser:
    us-phoenix-1
    us-ashburn-1
    sa-saopaulo-1
    entre outras

Por exemplo:

    $ fn update context api-url https://functions.us-ashburn-1.oci.oraclecloud.com

Configure o novo contexto com o endereço do registro e repositório que deseja usar com o Oracle Functions, digitando:

    $ fn update context registry <region-key>.ocir.io/<tenancy-namespace>/<repo-name>

Onde <region-key> é a mesma chave de três caracteres que você especificou ao fazer login no Oracle Cloud Infrastructure Registry, e <tenancy-namespace> é a string de namespace de armazenamento de objeto gerado automaticamente pela locação mostrada na página Informações de locação.

Por exemplo:

    $ fn update context registry phx.ocir.io/ansh81vru1zp/acme-repo

Agora você está pronto para começar a criar, implantar e invocar funções.



**T5.2.2.1.5 Criar, implantar e invocar sua função**

**T5.2.2.1.5.1 1Crie seu primeiro aplicativo**

No Oracle Functions , um aplicativo é:

* um agrupamento lógico de funções
* uma maneira de alocar e configurar recursos para todas as funções do aplicativo
* um contexto comum para armazenar variáveis de configuração que estão disponíveis para todas as funções no aplicativo
* uma maneira de garantir o isolamento do tempo de execução da função
	
Ao definir um aplicativo no Oracle Functions , você especifica as sub-redes nas quais executar as funções no aplicativo. Você também especifica se deseja habilitar o registro para as funções no aplicativo.
	
Quando funções de diferentes aplicativos são chamadas simultaneamente, o Oracle Functions garante que as execuções dessas funções sejam isoladas umas das outras.
A prática recomendada é agrupar várias funções em um único aplicativo para melhor eficiência e desempenho.
O Oracle Functions mostra os aplicativos e suas funções no console .
	
Faça login no console como desenvolvedor de funções e, em Desenvolvimento, clique em Funções e:

![menu_functions.png](https://github.com/hoshikawa2/repo-image/blob/master/menu_functions.png?raw=true)

![create_function.png](https://github.com/hoshikawa2/repo-image/blob/master/create_function.png?raw=true)


* Digite o nome de sua aplicação: **helloworld-app** 

* Selecione o compartimento especificado no contexto Fn Project CLI e que também foi criado para a sua VCN

* A VCN e a sub-rede (pública para efeitos didáticos) na qual irá executar a função.

* Você implantará sua primeira função neste aplicativo e especificará este aplicativo ao invocar a função.

* Clique em Criar 



**T5.2.2.1.6 Crie sua primeira função**
	
Em Oracle Functions , as funções são:

* blocos de código pequenos, mas poderosos, que geralmente fazem uma coisa simples
* agrupado em aplicativos
* armazenados como imagens Docker em um registro Docker especificado
* invocado em resposta a um comando CLI ou solicitação HTTP assinada
	
Quando você implanta uma função no Oracle Functions usando o Fn Project CLI, a função é construída como uma imagem Docker e enviada para um registro Docker especificado.
	
Uma definição da função é armazenada como metadados no servidor Oracle Functions . A definição descreve como a função deve ser executada e inclui:
	
* a imagem do Docker para puxar quando a função é invocada
* o período máximo de tempo que a função pode ser executada por
* a quantidade máxima de memória que a função pode consumir
	
Faça login em seu ambiente de desenvolvimento como desenvolvedor de funções e:

![Criar_primeira_funcao.png](https://github.com/hoshikawa2/repo-image/blob/master/Criar_primeira_funcao.png?raw=true)

Crie uma função java helloworld inserindo:

    $ fn init --runtime java helloworld-func

Um diretório chamado helloworld-func é criado, contendo:
• um arquivo de definição de função chamado func.yaml
• um diretório / src contendo arquivos de origem e diretórios para a função helloworld
• um arquivo de configuração Maven chamado pom.xml que especifica as dependências necessárias para compilar a função
Java é apenas uma das várias linguagens suportadas.



**T5.2.2.1.7 Implante sua primeira função**

Faça login em seu ambiente de desenvolvimento como desenvolvedor de funções e:

![Implante_primeira_funcao.png](https://github.com/hoshikawa2/repo-image/blob/master/Implante_primeira_funcao.png?raw=true)


Mude o diretório para o diretório helloworld-func criado na etapa anterior:

    $ cd helloworld-func

Insira o seguinte comando único Fn Project para construir a função e suas dependências como uma imagem Docker chamada helloworld-func, envie a imagem para o registro Docker especificado e implante a função no Oracle Functions no aplicativo helloworld-app que você criou anteriormente:

    $ fn -v deploy --app helloworld-app

(Opcional) Confirme se a imagem helloworld-func foi enviada para o Oracle Cloud Infrastructure Registry fazendo login no console como desenvolvedor de funções. Em Soluções e plataforma , vá para Serviços para desenvolvedores e clique em Registro . Escolha a região do registro e, em seguida, clique no nome do repositório especificado no contexto da CLI do projeto Fn para ver a função helloworld-func dentro dele.

![Pagina_de_registro.png](https://github.com/hoshikawa2/repo-image/blob/master/Pagina_de_registro.png?raw=true)

(Opcional) Confirme se a função foi implantada no Oracle Functions fazendo login no console como desenvolvedor de funções. Em Soluções e plataforma , vá para Serviços para desenvolvedores e clique em Funções . Selecione o compartimento especificado no contexto Fn Project CLI e, em seguida, clique em helloworld-app na página Aplicativos para ver se a função helloworld-func foi implantada no Oracle Functions.

![Pagina_funcoes.png](https://github.com/hoshikawa2/repo-image/blob/master/Pagina_funcoes.png?raw=true)



**T5.2.2.1.8 Invoque sua primeira função**
	
Em Oracle Functions , o código de uma função é executado (ou executado) quando a função é chamada (ou invocada). Você pode invocar uma função que implantou no Oracle Functions a partir de:
	
* O Projeto Fn CLI
* Os SDKs do Oracle Cloud Infrastructure 
* Solicitações HTTP assinadas para o endpoint de chamada da função. Cada função tem um endpoint invoke
* Outros serviços Oracle Cloud (por exemplo, acionados por um evento no serviço Eventos ) ou de serviços externos.
	
Quando uma função é chamada pela primeira vez, o Oracle Functions extrai a imagem Docker da função do registro Docker especificado, a executa como um contêiner do Docker e executa a função. Se houver solicitações subsequentes para a mesma função, o Oracle Functions direciona essas solicitações para o mesmo contêiner. Após um período de inatividade, o contêiner do Docker é removido.
	
O Oracle Functions mostra informações sobre invocações de função em gráficos de métricas.
	
Faça login em seu ambiente de desenvolvimento como desenvolvedor de funções e:

![fn_invoke.png](https://github.com/hoshikawa2/repo-image/blob/master/fn_invoke.png?raw=true)

Invoque a função helloworld-func no helloworld-app que você criou anteriormente inserindo:

    $ fn invoke helloworld-app helloworld-func

O 'Olá, mundo!' mensagem é exibida.

Invoque a função helloworld-func com o parâmetro 'John' inserindo:

![hello-world-fn.png](https://github.com/hoshikawa2/repo-image/blob/master/hello-world-fn.png?raw=true)

Janela de terminal com o comando fn invoke passando um parâmetro

    $ echo -n 'John' | fn invoke helloworld-app helloworld-func

O 'Olá, John!' mensagem é exibida.

Parabéns! Você acabou de criar, implantar e chamar sua primeira função usando Oracle Functions!

**T5.2.2.1.9 Saiba como invocar sua função com o CURL**

Neste tópico, iremos mostrar como montar sua chamada CURL para implementar nas mais diversas aplicações e cofigurações de eventos. 
Para isto, vamos utilizar um script Linux (infelizmente, este script não está disponível em Mac OS ou Windows, porém mostraremos ao final deste tópico como é uma chamada CURL típica.
	    
	    
Para invocar sua função via **curl**, baixe o arquivo bash **oci-curl.sh** deste tutorial.
Antes de mais nada, você deverá executar:

    chmod +x oci-curl.sh
    
Crie um arquivo chamado payload.json. Este arquivo deverá conter os parâmetros para o POST na chamada de uma função. Nosso caso não precisará de parâmetros, por isso, gere um arquivo em branco mesmo, desta forma:

    touch payload.json

Busque o endpoint de sua função na console da OCI navegando em:

* Menu principal
* Serviço do Desenvolvedor
* Functions
* Aplicações
* Selecione seu compartimento
* Clique na aplicação (helloworld-app)
* Copie o **Invoke endpoint**

![invoke_endpoint.png](https://github.com/hoshikawa2/repo-image/blob/master/invoke_endpoint.png?raw=true)

Seu endpoint pode ser dividido em 2 partes:

    https://5b2ey35uuyq.sa-saopaulo-1.functions.oci.oraclecloud.com/20181201/functions/ocid1.fnfunc.oc1.sa-saopaulo-1.aaaaaaaag5ainoruwqlq67t4uqizqmjjsa3os6jaxf3u5clo4f4iunzade6a/actions/invoke
    
    URL-para-funcao:
    5b2ey35uuyq.sa-saopaulo-1.functions.oci.oraclecloud.com --> será o primeiro parâmetro da chamada de seu oci-curl
    
    Funcao:
    /20181201/functions/ocid1.fnfunc.oc1.sa-saopaulo-1.aaaaaaaag5ainoruwqlq67t4uqizqmjjsa3os6jaxf3u5clo4f4iunzade6a/actions/invoke
     --> será seu segundo parâmetro do oci-curl

E assim que estiver pronto, você poderá executar:

    ./oci-curl.sh "<URL-para-funcao>" post payload.json "<Funcao>"
    
    Exemplo:
    ./oci-curl.sh "5b2ey35uuyq.sa-saopaulo-1.functions.oci.oraclecloud.com" post payload.json "/20181201/functions/ocid1.fnfunc.oc1.sa-saopaulo-1.aaaaaaaag5ainoruwqlq67t4uqizqmjjsa3os6jaxf3u5clo4f4iunzade6a/actions/invoke"

E com isto, você verá o comando **curl** montado como exemplo para você implementar em suas aplicações:

    curl --data-binary @payload.json -X POST -sS https://5b2ey35uuyq.sa-saopaulo-1.functions.oci.oraclecloud.com/20181201/functions/ocid1.fnfunc.oc1.sa-saopaulo-1.aaaaaaaag5ainoruwqlq67t4uqizqmjjsa3os6jaxf3u5clo4f4iunzade6a/actions/invoke -H 'date: Sun, 23 May 2021 14:06:27 GMT' -H 'x-content-sha256: 47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFU=' -H 'content-type: application/json' -H 'content-length: 0' -H 'Authorization: Signature version="1",keyId="ocid1.tenancy.oc1..aaaaaaaarwn3uriqeeoejrjjd7cj6sweix4dbjqe3o7c74zqsf4ojk7i4rwa/ocid1.user.oc1..aaaaaaaab4xpjyo2sxopwqvhwhhlejdqumfupzsrlh3ifnv3jcgtrj3w4riq/c5:4a:45:6f:ac:14:8b:d9:e9:30:1c:65:9e:ec:3f:0d",algorithm="rsa-sha256",headers="(request-target) date host x-content-sha256 content-type content-length",signature="bfHcsuCra0tgcPG3K454mg0WSJIWtpKnqIaGHl0aGI8Emjdy/8ayx9Fzlo9ObCLM72sgXtMmId5wnpWJJoyx8HCoE4FBfT8h/gaETeXd/5MvmB3rdp/5vGv0Yrk1mJ2074VUrFsQM8QVbi+7TPJ5srIzAhuvbc6Ms6NfO1Rep5RWjLW54+bF6rd8QQXG7TG1tvE4HE/+vh/auHbor2MaSl9oJVGIPQsV0WFOHdgvmujj1gC2QjRf857+7VyWdVDg4VHmCYFmMDHeXRKzG9HEvWVXXb1XbNMzCA4o0C5tGSonR1hn4ncOcCMhuOdtm6sDv4RryyUj2ASgOOudT5cw+A=="'

**T6 Próximos passos**

Agora que você criou, implantou e invocou uma função, leia a documentação para descobrir como:

* exportar registros de funções configurando um URL de syslog ou enviando registros para um depósito de armazenamento (consulte Armazenamento e visualização de registros de funções )
* explorar Oracle Functions usando amostras no GitHub (consulte Oracle Functions Samples )
* invocar uma função usando SDKs (consulte Usando SDKs para invocar funções )

----

Referências:

Oracle Container Registry - OCIR
- https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/index.html
- https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm#two

Oracle Functions - Quick-View (base do material utilizado neste workshop)
- https://www.oracle.com/webfolder/technetwork/tutorials/infographics/oci_faas_gettingstarted_quickview/functions_quickview_top/functions_quickview/index.html

Oracle Functions - Samples
- https://github.com/oracle/oracle-functions-samples/tree/master/samples

Oracle Functions - Connecting To ATP With Node.JS
- https://blogs.oracle.com/developers/oracle-functions-connecting-to-atp-with-nodejs

Oracle API Gateway - Deploying a function in API Gateway
- https://blogs.oracle.com/developers/creating-your-first-api-gateway-in-the-oracle-cloud

Deploy Oracle functions with DevOps
- https://docs.oracle.com/en/cloud/paas/visual-builder/tutorial-create-project-upload/index.html
- https://blogs.oracle.com/developers/cicd-automation-for-project-fn-with-oracle-faas-and-developer-cloud-service

Oracle Cloud - Creating an Oracle Linux VM
- https://blogs.oracle.com/lad-cloud-experts/pt/como-criar-uma-virtual-machine-utilizando-o-oracle-cloud-infrastructure

Oracle Cloud - Logging and Functions
- https://docs.oracle.com/pt-br/iaas/Content/Functions/Tasks/functionsexportingfunctionlogfiles.htm
	    
