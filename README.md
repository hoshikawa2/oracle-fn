# OCI DEV Functions

O objetivo deste workshop é permitir ao participante que deseja conhecer um pouco do mundo de Serverless em cloud com functions.

Em nosso dia-a-dia, ouvimos muito sobre estes termos mas muitas vezes, entender cada conceito se torna algo bastante complexo. Certo?

O workshop irá compreender os seguintes conceitos:

- Cloud
- Repositório de Imagens Docker
- Serverless 
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
    

### T1.4 Instalando o Docker

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
    
**T1.4.1 Linux**

Para instalar o docker no Oracle Linux, CentOS, Redhat, primeiramente faça o setup do repositório:

    $ sudo yum install -y yum-utils

    $ sudo yum-config-manager --add-repo https://download.docker.com/linux/<edição do Linux>/docker-ce.repo
    
    Edições de Linux disponíveis:
    centos
    debian
    fedora
    raspbian
    rhel
    static
    ubuntu
    
    exemplo para rhel:
    $ sudo yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

Em seguida, instale a última versão do Docker:

    $ sudo yum install docker-ce docker-ce-cli containerd.io --skip-broken

Será apresentado o fingerprint conforme abaixo. Assim que for apresentado, clique em aceitar.

    060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35
    
Inicie o docker com:

    $ sudo systemctl start docker
    
Teste para saber se o docker está funcionando:

    $ sudo docker run hello-world
    

**T1.4.2 Ubuntu**

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
    
**T1.4.3 Mac OS X**

Faça o download na página do docker para Mac OS X:

    https://hub.docker.com/editions/community/docker-ce-desktop-mac/
    
Dê duplo-clique com o mouse no arquivo Docker.dmg e siga os passos.

Na janela do terminal, teste o docker com o comando:

    $ sudo docker run hello-world

**T1.4.4 Windows**

Faça o download na página do docker para Mac OS X:

    https://hub.docker.com/editions/community/docker-ce-desktop-windows/
    
Dê duplo-clique com o mouse no arquivo "Docker for Windows Installer" e siga os passos.

Na janela do Powershell, teste o docker com o comando:

    $ docker run hello-world
    

### T1.5 Como gerar uma chave de assinatura de API

    Obs: A etapa T1.5 pode ser feita automaticamente na instalação do OCI CLI. Apenas para conhecimento, entenda que o processo manual de gerar as chaves pública e privada é feita da forma descrita aqui em T1.5. Pule para a etapa T2 para ganhar tempo e gere as chaves automaticamente.


**T1.5.1 - Gerando uma chave de assinatura de API (Linux e Mac OS X)**

Use os seguintes comandos OpenSSL para gerar o par de chaves no formato PEM necessário.

**T1.5.1.1** Crie um .ocidiretório para armazenar as credenciais, caso ainda não o tenha feito :


    mkdir ~/.oci                

**T1.5.1.2** Gere a chave privada com um dos seguintes comandos.

Para gerar a chave, criptografada com uma senha longa que você fornece quando solicitado:
 
    Observação

    Recomendamos que você use uma senha longa para sua chave.
    
    openssl genrsa -out ~/.oci/oci_api_key.pem -aes128 2048       
                 

Para gerar a chave sem senha:

    openssl genrsa -out ~/.oci/oci_api_key.pem 2048                        
    
**T1.5.1.3** Altere a permissão do arquivo para garantir que apenas você possa ler o arquivo da chave privada:

    chmod go-rwx ~/.oci/oci_api_key.pem               
    
**T1.5.1.4** Gere a chave pública a partir de sua nova chave privada:

    openssl rsa -pubout -in ~/.oci/oci_api_key.pem -out ~/.oci/oci_api_key_public.pem             
    
**T1.5.1.5** Copie o conteúdo da chave pública para a área de transferência usando pbcopy, xclip ou uma ferramenta semelhante (você precisará colar o valor no console posteriormente). Por exemplo:

    cat ~/.oci/oci_api_key_public.pem | pbcopy           
    
Suas solicitações de API serão assinadas com sua chave privada e a Oracle usará a chave pública para verificar a autenticidade da solicitação. Você deve carregar a chave pública para o IAM (instruções abaixo).

**T1.5.2 - Gerando uma chave de assinatura de API (Windows)**

Se estiver usando o Windows, você precisará instalar o Git Bash para Windows antes de executar os comandos a seguir.

    Observação

    Certifique-se de incluir o openssl binário no caminho do Windows. Na instalação
    padrão, o arquivo openssl.exe pode ser encontrado em 
    C:\Program Files\Git\mingw64\bin.
    
Use os seguintes comandos OpenSSL para gerar o par de chaves no formato PEM necessário.

**T1.5.2.1** Crie um .ocidiretório para armazenar as credenciais, caso ainda não o tenha feito . Por exemplo:

    mkdir %HOMEDRIVE%%HOMEPATH%\.oci                
    
**T1.5.2.2** Gere a chave privada com um dos seguintes comandos:

Para gerar a chave que é criptografada com uma senha longa que você fornece quando solicitado:
 
    Observação
    Recomendamos que você use uma senha longa para sua chave.

    openssl genrsa -out %HOMEDRIVE%%HOMEPATH%\.oci\oci_api_key.pem -aes128 -passout stdin 2048
                     
Para gerar a chave sem senha:

    openssl genrsa -out %HOMEDRIVE%%HOMEPATH%\.oci\oci_api_key.pem 2048                        

**T1.5.2.3** Gere a chave pública a partir de sua nova chave privada:

    openssl rsa -pubout -in %HOMEDRIVE%%HOMEPATH%\.oci\oci_api_key.pem -out %HOMEDRIVE%%HOMEPATH%\.oci\oci_api_key_public.pem             

**T1.5.2.4** Copie o conteúdo da chave pública para a área de transferência (você precisará colar o valor no console posteriormente). Por exemplo:

    type \.oci\oci_api_key_public.pem     
    
Suas solicitações de API serão assinadas com sua chave privada e a Oracle usará a chave pública para verificar a autenticidade da solicitação. Você deve carregar a chave pública para o IAM (instruções abaixo).

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

    Obs: Na etapa T1.4, mostramos como gerar as chaves pública e privada. Se você 
    optou por gerá-las automaticamente, é aqui que faremos isso. Ao executar o comando
    oci setup config, você será questionado se deseja carregar suas chaves previamente
    geradas ou se deseja gerar as chaves.
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
- Escolha a região em que você estará trabalhando (por exemplo, us-phoenix-1). 
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
    
    ![Helloworld por Karthquian](https://camo.githubusercontent.com/b13f75f4886d7994faf2aa86b438710c3a61f8a5032568f6f02828de59ab3220/68747470733a2f2f7777772e6f7261636c652e636f6d2f776562666f6c6465722f746563686e6574776f726b2f7475746f7269616c732f6f62652f6f63692f72656769737472792f696d672f6f63692d696d6167652d726561646d652d636f6d706c6574652e706e67)


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

**T5.1 Configure sua locação**

**T5.1.1 Crie grupos e usuários**

Caixa de diálogo Criar Grupo
Se os usuários e grupos adequados ainda não existirem, faça login no console como administrador de locação e, em Governança e administração , acesse Identidade e:

Crie um novo grupo clicando em Grupos e depois em Criar Grupo .

![create_group.png](https://github.com/hoshikawa2/repo-image/blob/master/create_group.png?raw=true)


Crie um novo usuário clicando em Usuários e em Criar usuário. Lembre-se de selecionar "IAM User". Após criar o usuário, este usuário irá receber um e-mail para ajustar sua senha.

![criar_usuario2.png](https://github.com/hoshikawa2/repo-image/blob/master/criar_usuario2.png?raw=true)

Adicione um usuário a um grupo clicando em Grupos , depois no nome do grupo e em Adicionar usuário ao grupo .

![adicionar_usuario_grupo.png](https://github.com/hoshikawa2/repo-image/blob/master/adicionar_usuario_grupo.png?raw=true)



**T5.1.2 Criar compartimento**

**T5.1.2.1 Criar página de compartimento**

Se ainda não houver um compartimento adequado para criar recursos de rede e recursos de funções Oracle, faça login no console como administrador de locação e, em Governança e administração , acesse Identidade e:

Clique em Compartments e em Create Compartment.

![Criar_compartment.png](https://github.com/hoshikawa2/repo-image/blob/master/Criar_compartment.png?raw=true)




**T5.1.3 Crie VCN e sub-redes**

**T5.1.3.1 Iniciar caixa de diálogo do assistente VCN**

Se ainda não houver um VCN adequado para criar recursos de rede, faça login no console como administrador de locação e em Infraestrutura central , vá para Rede e, em seguida:

* Clique em Redes e em VCN (Virtual Cloud Network) e escolha um compartimento.
* Clique em Start VCN Wizard , depois em VCN with Internet Connectivity e em Start VCN Wizard .
* Insira um nome para o novo VCN, clique em Avançar e em Criar para criar o VCN junto com os recursos de rede relacionados.


![start_vcn_wizard.png](https://github.com/hoshikawa2/repo-image/blob/master/start_vcn_wizard.png?raw=true)





**T5.1.4 Criar política para o grupo**

Caixa de diálogo Criar Política

* Faça login no console como administrador de locação e, em Governança e administração , acesse Identidade e clique em Políticas e, em seguida:

![Criar_politica.png](https://github.com/hoshikawa2/repo-image/blob/master/Criar_politica.png?raw=true)

* Se um ou mais usuários do Oracle Functions não for um administrador de locação, use o Criador de políticas para criar uma nova política. Selecione o compartimento raiz da locação e baseie a política no modelo de política. Permitir que os usuários criem, implantem e gerenciem funções e aplicativos usando o Cloud Shell .
O modelo de política inclui as seguintes declarações de política:

    Allow group <group-name> to use cloud-shell in tenancy
    Allow group <group-name> to manage repos in tenancy
    Allow group <group-name> to read objectstorage-namespaces in tenancy
    Allow group <group-name> to read metrics in tenancy
    Allow group <group-name> to manage functions-family in tenancy
    Allow group <group-name> to use virtual-network-family in tenancy
    
* Se necessário, você pode restringir essas declarações de política por compartimento (conforme descrito na documentação ).

**T5.2 Configure seu ambiente de desenvolvimento**

**T.5.2.1 Instalar Fn Project CLI**

Faça login em seu ambiente de desenvolvimento como desenvolvedor de funções e:
marca de verificação

Abra o arquivo Fn Project CLI README.md no GitHub e siga as instruções apropriadas para instalar o Fn Project CLI. Como uma visão geral conveniente, as instruções são resumidas abaixo:

    Linux ou MacOS:
    $ curl -LSs https://raw.githubusercontent.com/fnproject/cli/master/install | sh
    
    MacOS usando Homebrew:
    $ brew install fn
    
    Linux, MacOS ou Windows: baixe e execute o binário da página Fn Project Releases no GitHub
    https://github.com/fnproject/cli/releases

Confirme se a CLI foi instalada inserindo:

    $ fn version



**T5.2.2.2 Configure Fn Project CLI no ambiente de desenvolvimento local**

**T5.5.2.2.1 Configurar chave de assinatura**

* Faça login em seu ambiente de desenvolvimento de máquina local como desenvolvedor de funções e:

Gere uma chave privada criptografada com uma senha longa que você fornece inserindo:

    $ openssl genrsa -out ~/.oci/<private-key-file-name>.pem -aes128 2048

Altere as permissões no arquivo para garantir que somente você possa lê-lo.

    $ chmod go-rwx ~/.oci/<private-key-file-name>.pem

Gere uma chave pública (criptografada com a mesma senha longa que você forneceu ao criar a chave privada e no mesmo local do arquivo da chave privada) inserindo:

    $ openssl rsa -pubout -in ~/.oci/<private-key-file-name>.pem -out ~/.oci/<public-key-file-name>.pem

Copie o conteúdo do arquivo de chave pública que você acabou de criar, digitando:

    $ cat ~/.oci/<public-key-file-name>.pem | pbcopy

Criar chave API
![Chave_API.png](https://github.com/hoshikawa2/repo-image/blob/master/Chave_API.png?raw=true)

Faça login no console como desenvolvedor de funções, abra o menu do usuário ( Ícone do menu do usuário) e vá para as configurações do usuário . Na página Chaves de API , clique em Adicionar chave pública . Cole o valor da chave pública na janela e clique em Adicionar . A chave é carregada e sua impressão digital é exibida.



**T5.2.2.2.2 Configurar perfil OCI**

Faça login em seu ambiente de desenvolvimento de máquina local como desenvolvedor de funções e:
marca de verificação

Abra o arquivo ~ / .oci / config em um editor de texto. (Se o diretório e / ou o arquivo ainda não existirem, crie-os).
marca de verificação

Adicione um novo perfil ao arquivo ~ .oci / config da seguinte maneira:

    [<profile-name>]
    user=<user-ocid>
    fingerprint=<public-key-fingerprint>
    key_file=<full-path-to-private-key-pem-file>
    tenancy=<tenancy-ocid>
    region=<region-name>
    pass_phrase=<passphrase>

Salve e feche o arquivo.



**T5.2.2.2.3 Configure o contexto CLI do projeto Fn - oráculo do provedor** 

Faça login em seu ambiente de desenvolvimento de máquina local como desenvolvedor de funções e:

Crie um novo contexto Fn CLI inserindo:

    $ fn create context <my-context> --provider oracle

Observe que você especifica --provider oraclea ativação da autenticação e autorização usando assinatura de solicitação do Oracle Cloud Infrastructure, chaves privadas, grupos de usuários e políticas que concedem permissões a esses grupos de usuários.
marca de verificação

Especifique que a CLI do projeto Fn deve usar o novo contexto inserindo:

    $ fn use context <my-context>

Configure o novo contexto com o nome do perfil OCI que você criou para usar com o Oracle Functions, digitando:

    $ fn update context oracle.profile <profile-name>



**T5.2.2.2.4 Concluir a configuração do contexto CLI do projeto Fn**

Faça login em seu ambiente de desenvolvimento como desenvolvedor de funções e:
marca de verificação

Configure o novo contexto com o OCID do compartimento em que deseja possuir as funções implantadas:

    $ fn update context oracle.compartment-id <compartment-ocid>

Configure o novo contexto com o endpoint api-url para usar ao chamar a API OCI inserindo:

    $ fn update context api-url <api-endpoint>

Por exemplo:

    $ fn update context api-url https://functions.us-phoenix-1.oci.oraclecloud.com

Configure o novo contexto com o endereço do registro e repositório que deseja usar com o Oracle Functions, digitando:

    $ fn update context registry <region-key>.ocir.io/<tenancy-namespace>/<repo-name>

Onde <region-key> é a mesma chave de três caracteres que você especificou ao fazer login no Oracle Cloud Infrastructure Registry, e <tenancy-namespace> é a string de namespace de armazenamento de objeto gerado automaticamente pela locação mostrada na página Informações de locação.

Por exemplo:

    $ fn update context registry phx.ocir.io/ansh81vru1zp/acme-repo

Agora você está pronto para começar a criar, implantar e invocar funções.



**T5.2.2.2.5 Criar, implantar e invocar sua função**

**T5.2.2.2.5.1 1Crie seu primeiro aplicativo**

Faça login no console como desenvolvedor de funções e, em Soluções e plataforma , acesse Serviços para desenvolvedores e clique em Funções e:

![Nova_aplicacao.png](https://github.com/hoshikawa2/repo-image/blob/master/Nova_aplicacao.png?raw=true)


Selecione a região que pretende usar para o Oracle Functions (recomendado que seja a mesma região do registro Docker especificado no contexto Fn Project CLI).

Selecione o compartimento especificado no contexto Fn Project CLI.

Clique em Criar aplicativo e especifique:

**helloworld-app** como o nome do novo aplicativo. Você implantará sua primeira função neste aplicativo e especificará este aplicativo ao invocar a função.
A VCN e a sub-rede na qual executar a função.

Clique em Criar .



**T5.2.2.2.6 Crie sua primeira função**

Faça login em seu ambiente de desenvolvimento como desenvolvedor de funções e:
marca de verificação

![Criar_primeira_funcao.png](https://github.com/hoshikawa2/repo-image/blob/master/Criar_primeira_funcao.png?raw=true)

Crie uma função java helloworld inserindo:

    $ fn init --runtime java helloworld-func

Um diretório chamado helloworld-func é criado, contendo:
• um arquivo de definição de função chamado func.yaml
• um diretório / src contendo arquivos de origem e diretórios para a função helloworld
• um arquivo de configuração Maven chamado pom.xml que especifica as dependências necessárias para compilar a função
Java é apenas uma das várias linguagens suportadas.



**T5.2.2.2.7 Implante sua primeira função**

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



**T5.2.2.2.8 Invoque sua primeira função**

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



**T6 Próximos passos**

Agora que você criou, implantou e invocou uma função, leia a documentação para descobrir como:

• exportar registros de funções configurando um URL de syslog ou enviando registros para um depósito de armazenamento (consulte Armazenamento e visualização de registros de funções )
• explorar Oracle Functions usando amostras no GitHub (consulte Oracle Functions Samples )
• invocar uma função usando SDKs (consulte Usando SDKs para invocar funções )

----

Referências:

- https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/index.html
- https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm#two

- https://www.oracle.com/webfolder/technetwork/tutorials/infographics/oci_faas_gettingstarted_quickview/functions_quickview_top/functions_quickview/index.html
