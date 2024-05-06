## Atividade **Docker** DevSecOps Compass UOL
A atividade envolveu a configuração de um ambiente na AWS com Docker para implantar o WordPress. Os requisitos incluíam instalação do Docker em uma instância EC2 via script, implantação do WordPress com MySQL em contêineres, uso do serviço EFS para armazenamento estático, configuração de um Load Balancer Classic para escalabilidade e alta disponibilidade, evitando o uso de IPs públicos e demonstrando a aplicação WordPress em funcionamento.

### Requisitos:
> [!IMPORTANT]
> - Instalação e configuração do DOCKER ou CONTAINERD no host EC2.
> - Ponto adicional para o trabalho: Utilizar a instalação via script de Start Instance (user_data.sh).
> - Efetuar deploy de uma aplicação WordPress com container de aplicação RDS database MySQL.
> - Configuração da utilização do serviço EFS AWS para estáticos do container de aplicação WordPress.
> - Configuração do serviço de Load Balancer AWS para a aplicação WordPress.

### Pontos de atenção:
- Não utilizar IP público para saída dos serviços WordPress (Evitem publicar o serviço WordPress via IP público).
- Sugestão para o tráfego: Internet sair pelo LB (Load Balancer Classic).
- Pastas públicas e estáticos do WordPress sugestão de utilizar o EFS (Elastic File System).
- Fica a critério de cada integrante usar Dockerfile ou Docker Compose.
- Necessário demonstrar a aplicação WordPress funcionando (tela de login).
- Aplicação WordPress precisa estar rodando na porta 80 ou 8080.
- Utilizar repositório git para versionamento.
- Criar documentação.

## Etapas de execução

### Configuração da Network:
- Acessei o console AWS e entrei no serviço **VPC**.
- No menu lateral esquerdo, na seção de **NUVEM PRIVADA VIRTUAL** selecionei **SUAS VAPCs**.
- Dentro de **SUAS VPCs** cliquei no botão **CRIAR VPC**.
- Alterei as seguintes configurações:
    - Em **RECURSOS A SEREM CRIADOS** selecionei **VPC E MUITO MAIS**.
    - Em **TAG DE NOME** coloquei o nome **atividade2**.
    - Em **NÚMERO DE ZONAS DE DISPONIBILIDADE (AZs)** selecionei **2**.
    - Em **GATEWAYS NAT** selecionei **EM 1 AZ**.
    - Em **ENDPOINTS DA VPC** selecionei **NENHUMA**.
- Cliquei em **CRIAR VPC**.

![image](https://github.com/AlonsoNeto01/Atividade_Docker_Compass-UOL_AWS/assets/164195128/18f16f51-0d73-4b2f-af2b-f5588045786e)


### Configuração dos Security Groups:
- Acessei o console AWS e entrei no serviço **EC2**.
- No menu lateral esquerdo, na seção de **REDE E SUGURANÇA**, selecionei **SECURITY GROUPS**.
- Dentro de **SECURITY GROUPS**, cliquei no botão **CRIAR GRUPO DE SUGURANÇA**.
- Criei e configurei os seguintes grupos usando a VPC criada anteriormente:

    - #### Load Balancer - Regras de entrada
        | Type | Protocol | Port Range |   Source  |
        |:----:|:--------:|:----------:|:---------:|
        | HTTP | TCP      | 80         | 0.0.0.0/0 |

    - #### EC2_WS - Regras de entrada
        | Type | Protocol | Port Range |       Source       |
        |:----:|:--------:|:----------:|:------------------:|
        |  SSH |    TCP   |     22     |        EC2         |
        | HTTP |    TCP   |     80     |    Load Balancer   |

    - #### EC2 - Regras de saída
        | Type | Protocol | Port Range |       Source       |
        |:----:|:--------:|:----------:|:------------------:|
        |  SSH |    TCP   |     22     |       EC2_WS       |

    - #### Amazon RDS - Regras de entrada
        |     Type     | Protocol | Port Range |        Source       |
        |:------------:|:--------:|:----------:|:-------------------:|
        | MYSQL/Aurora |    TCP   |    3306    |        EC2_WS       |

    - #### EFS - Regras de entrada
        | Type | Protocol | Port Range |        Source       |
        |:----:|:--------:|:----------:|:-------------------:|
        | NFS  | TCP      | 2049       |        EC2_WS       |

      

### Criando o EFS:
- Acessei o console AWS e entrei no serviço de **EFS**.
- Na tela do **Elastic File System** cliquei no botão **CRIAR SISTEMA DE ARQUIVOS**.
- Depois cliquei no botão **PERSONALIZAR**.
- Executei a seguinte configuração: 

    - #### ETAPA 1 - Configurações do sistema de arquivos:
        - No campo **NOME** digitei "ATIVIDADE_2".
        - Cliquei em **PRÓXIMO**.

    - #### ETAPA 2 - Acesso à rede:
        - No campo **Virtual Private Cloud (VPC)** selecionei a VPC que foi criada anteriormente.
        - No campo **ID da sub-rede** selecionei as subnets privadas de cada AZ.
        - No campo **Grupos de segurança** selecionei o grupo "EFS" que foi criado anteriormente.
        - Cliquei em **PRÓXIMO**.

    - #### ETAPA 3 - opcional - Política do sistema de arquivos:
        - Cliquei em **PRÓXIMO**.
        
    - #### ETAPA 4 - Revisar e criar:
        - Revisei e cliquei em **CRIAR** para finalizar.

### Criando o Relational Database Service:
- Acessei o console AWS e entrei no serviço de **RDS**.
- Na tela **PAINEL** cliquei no botão **Criar banco de dados**.
- Executei a seguinte configuração:
    - Na seção **Opções do mecanismo** selecionei **MySQL**.
    - Na seção **Modelos** selecionei **Nível gratuito**.
    - Na seção **Configurações de credenciais** fui em **Autogerenciada** e criei a senha.
    - Na seção **Conectividade**, no campo **Nuvem privada virtual (VPC)** selecionei a VPC criada anteriormente.
    - No campo **Grupos de segurança da VPC existentes** selecionei o grupo "Amazon RDS" que foi criado anteriormente.
    - Na seção **Configuração adicional**, no campo **Porta do banco de dados** coloquei o nome "TDB".
- Revisei e cliquei em **Criar banco de dados** para finalizar.

### Criando o Classic Load Balancer:
- Acessei o console AWS e entrei no serviço **EC2**.
- No menu lateral esquerdo, na seção de **Balanceamento de carga** selecionei **Load Balancers**.
- Dentro de **Load Balancers** cliquei no botão **Criar load balancer**.
- Em **Tipos de load balancer** cliquei em **Classic Load Balancer - geração anterior** e depois em **Criar**.
- No campo **Nome do load balancer** digitei "LB".
- Na seção **Mapeamento de rede**, no campo **VPC** selecionei a VPC criada anteriormente.
- No campo **Mapeamentos** selecionei as duas AZ's e suas respectivas subnets públicas.
- No campo de **Grupos de segurança** selecionei o grupo "Load Balancer" que foi criado anteriormente.
- Na seção **Verificações de integridade**, no campo **Caminho de ping** adicionei o caminho "/wp-admin/install.php".
- Cliquei em **Criar load balancer** para finalizar.

  ![Captura de tela 2024-05-05 155432](https://github.com/AlonsoNeto01/Atividade_Docker_Compass-UOL_AWS/assets/164195128/0fba7b04-f9c5-4565-9130-bd9bf4c58803)



### Gerando a Key pair:
- Acessei o console AWS e entrei no serviço **EC2**.
- No menu lateral esquerdo, na seção de **Rede de sugurança** selecionei **Pares de chaves**.
- Dentro de **Pares de chaves** cliquei no botão **Criar par de chaves**.
- No campo **Nome** digitei "Atividade_2". 
- No campo **tipo de par de chaves** selecionei **RSA**.
- No campo **Formato de arquivo de chave privada** selecionei **.pem**.
- Cliquei no botão **Criar par de chaves**.
- Salvei o arquivo .pem.



### Criando o Launch Template:
- Acessei o console AWS e entrei no serviço **EC2**.
- No menu lateral esquerdo, na seção **Instâncias** selecionei **Modelos de execução**.
- Dentro de **Modelos de execução** cliquei no botão **Criar modelo de execução**.
- No campo **Nome e descrição do modelo de execução** digitei "Atividade-2".
- No campo **Template version description** digitei "docker-wordpress".
- Em **Imagens de aplicação e de sistema operacional** cliquei em **Procurar mais AMIs**, depois cliquei em **Amazon Linux** e selecionei a Amazon Linux 2023 AMI.
- Na seção **Tipo de instância** selecionei o tipo **t3.small**.
- No campo **Par de chaves** selecionei a chave criada anteriormente.
- Em **Configurações de rede**, no campo **Grupos de segurança** selecionei o grupo "EC2_WS" que foi criado anteriormente.
- Em **Tags de recurso** cliquei em **Adicionar tags** e adicione as tags conforme foi orientado.
- Em **Detalhes avançados**, no campo **Dados do usuario** adicionei o script abaixo:
    ```
   #!/bin/bash
   #Atualizar os pacotes do sistema
   sudo yum update -y

   #Instalar, iniciar e configurar a inicialização automática do docker
   sudo yum install docker -y 
   sudo systemctl start docker
   sudo systemctl enable docker

   #Adicionar o usuário ec2-user ao grupo docker
   sudo usermod -aG docker-EC2 ec2-user

   #Instalação do docker-compose
   sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose

   #Instalar, iniciar e configurar a inicialização automática do nfs-utils
   sudo yum install nfs-utils -y
   sudo systemctl start nfs-utils
   sudo systemctl enable nfs-utils

   #Criar a pasta onde o EFS vai ser montado
   sudo mkdir /mnt/efs

   #Montagem e configuração da montagem automática do EFS
   sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-09b515a414fc862ba.efs.us-east-1.amazonaws.com:/ /mnt/efs
   sudo echo "fs-09b515a414fc862ba.efs.us-east-1.amazonaws.com:/ /mnt/efs nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport,_netdev 0 
   0" >> /etc/fstab

   # Criar uma pasta para os arquivos do WordPress
   sudo mkdir /mnt/efs/wordpress

   # Criar um arquivo docker-compose.yaml para configurar o WordPress
   sudo cat <<EOL > /mnt/efs/docker-compose.yaml
   version: '3.8'
   services:
     wordpress:
       image: wordpress:latest
       container_name: wordpress
       ports:
         - "80:80"
       environment:
         WORDPRESS_DB_HOST: database-1.cjkoeko6wclb.us-east-1.rds.amazonaws.com
         WORDPRESS_DB_USER: admin
         WORDPRESS_DB_PASSWORD: alonso00
         WORDPRESS_DB_NAME: TDB
         WORDPRESS_TABLE_CONFIG: wp_
       volumes:
         - /mnt/efs/wordpress:/var/www/html
   EOL

   # Inicializar o WordPress com docker-compose
   docker-compose -f /mnt/efs/docker-compose.yaml up -d
    ```
- Cliquei em **Criar modelo de execução** para finalizar.

### Criando o Auto Scaling Groups:
- Acessei o console AWS e entrei no serviço **EC2**.
- No menu lateral esquerdo, na seção de **Auto Scaling** selecionei **Grupos de Auto Scaling**.
- Dentro de **Grupos de Auto Scaling** cliquei no botão **Criar grupo de Auto Scaling**.
- Executei a seguinte configuração:
    - #### Etapa 1 - Escolher o modelo de execução:
        - No campo **Nome do grupo do Auto Scaling** digitei "Auto_Scaling".
        - Na seção **Modelo de execução** selecionei o template criado anteriormente.
        - Cliquei em **Próximo**.
    - #### Etapa 2 - Escolher as opções de execução de instância:
        - Na seção **Rede**, no campo **VPC** selecionei a VPC criada anteriormente.
        - No campo **Zonas de disponibilidade e sub-redes** selecionei as duas subnets privadas criadas previamente.
        - Cliquei em **Próximo**.
    - #### Etapa 3 - Configurar opções avançadas - opcional :
        - Na seção **Balanceamento de carga ** selecionei **Anexar a um balanceador de carga existente**.
        - Na seção **Anexar a um balanceador de carga existente** cliquei em **Escolher entre Classic Load Balancers** e selecionei o load balancer criado anteriormente.
        - Na seção **Verificações de integridade** marquei a opção **Ative as verificações de integridade do Elastic Load Balancing**.
        - Cliquei em **Próximo**.
    - #### Etapa 4 - Configurar tamanho do grupo e ajuste de escala - opcional :
        - No campo **Tamanho do grupo ** digitei "2".
        - Em **Escalabilidade**, no campo **Capacidade mínima desejada** digitei "2".
        - No campo **Capacidade máxima desejada** digitei "4".
        - Em **Ajuste de escala automática - opcional** selecionei a opção **Política de dimensionamento com monitoramento do objetivo**
        - No campo **Tipo de métrica** deixei selecionado **Média de utilização da CPU**.
        - No campo **Target value** digitei "75".
        - Cliquei em **Próximo**.
    - #### Etapa 5, 6 e 7:
        - Cliquei em **Próximo**.
        - Cliquei em **Próximo**.
        - Revisei e cliquei em **Criar grupo de Auto Scaling** para finalizar.

### Configuração do EC2 Instance Connect Endpoint:
- Acessei o console AWS e entrei no serviço **VPC**.
- No menu lateral esquerdo, na seção de **Nuvem privada virtual** selecionei **Endpoints**.
- Dentro de **Endpoints** cliquei no botão **Criar endpoint**.
- Alterei as seguintes configurações:
    - Em **Etiqueta de nome** coloquei o nome "endpoint".
    - Em **Categoria de serviço** selecionei **Endpoint do EC2 Instance Connect**.
    - Em **VPC** selecionei a VPC criada anteriormente.
    - Em **Grupos de segurança** selecionei o grupo "EC2" que foi criado anteriormente.
    - Em **Subnet** selecionei uma subnet privada que foi criada anteriormente.
- Cliquei em **Criar endpoint**.

![Captura de tela 2024-05-05 192153](https://github.com/AlonsoNeto01/Atividade_Docker_Compass-UOL_AWS/assets/164195128/f0056a4a-f9d1-439c-8d5f-d35b84c32158)



### Instalando o WordPress:
- Acessei o **DNS name** do **Load Balancer** através do navegador.
- Na tela de instalação do **WordPress** mantive o idioma padrão e cliquei em **Continue**.

  ![Captura de tela 2024-05-05 163405](https://github.com/AlonsoNeto01/Atividade_Docker_Compass-UOL_AWS/assets/164195128/5055aa78-e808-4f1b-a8d2-9a7e556e5802)


- Na tela seguinte preenchi os dados para criação de um usuário.

![Captura de tela 2024-05-05 163535](https://github.com/AlonsoNeto01/Atividade_Docker_Compass-UOL_AWS/assets/164195128/71ec75fc-a172-4ade-978e-ef9f9508a9a4)


- Cliquei em **Install WordPress** para finalizar.

 ![Captura de tela 2024-05-05 163730](https://github.com/AlonsoNeto01/Atividade_Docker_Compass-UOL_AWS/assets/164195128/ef537f6e-574a-4b49-a663-67fcaf53683d)

Para as referências usadas na condução da atividade, consulte a documentação oficial:
https://hub.docker.com/_/wordpress
[https://docs.aws.amazon.com/pt_br/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.html](https://docs.aws.amazon.com/pt_br/)
https://docs.docker.com/compose/install/standalone/
https://github.com/docker

 ![image](https://github.com/AlonsoNeto01/Atividade_Docker_Compass-UOL_AWS/assets/164195128/cc77143f-f81b-4e5b-a469-040d16f22be8)
