## ☁️ ARQUITETURA DO PROJETO

- **Auto Scaling Group (ASG)** com múltiplas instâncias EC2 para executar o WordPress.  
- **Application Load Balancer (ALB)** para balancear o tráfego de entrada.  
- **Amazon EFS** para armazenamento compartilhado de arquivos do WordPress (mídias, plugins, etc.).  
- **Amazon RDS (MySQL/MariaDB)** para armazenar dados como posts e informações de usuários.

---

## 🛠️ COMPONENTES NA AWS

### 🔹 VPC Personalizada
- **Subnets**: 2 públicas e 4 privadas, distribuídas em duas Zonas de Disponibilidade (AZs)  
- **Internet Gateway (IGW)**: Permite acesso à internet pelas subnets públicas  
- **NAT Gateway**: Localizado nas subnets públicas, permitindo que as instâncias privadas acessem a internet  

---

## 🚀 ETAPAS DE IMPLEMENTAÇÃO

### 1. Criar a VPC
- Criar 2 subnets públicas  
- Criar 4 subnets privadas  
- Criar IGW e NAT Gateway
<img width="1404" height="880" alt="Image" src="https://github.com/user-attachments/assets/14954835-7c4d-49b8-bc43-909385cf85ed" />

### 2. Criando Security Groups
- No painel da EC2, no canto esquerdo no grupo Rede e Segurança, clique em Security Groups (Grupos de Segurança)
- Crie os seguintes grupos:
  
#### sg-ALB (Application Load Balancer)
| Direção | Tipo | Protocolo | Porta | Origem/Destino |
|---------|------|-----------|-------|----------------|
| Entrada | HTTP | TCP       | 80    | 0.0.0.0/0      |
| Saída   | HTTP | TCP       | 80    | sg-EC2         |

#### sg-RDS (Banco de Dados MySQL/MariaDB)
| Direção | Tipo         | Protocolo | Porta | Origem/Destino |
|---------|--------------|-----------|-------|----------------|
| Entrada | MySQL/Aurora | TCP       | 3306  | sg-EC2         |
| Saída   | MySQL/Aurora | TCP       | 3306  | sg-EC2         |

#### sg-EFS (Armazenamento EFS)
| Direção | Tipo | Protocolo | Porta | Origem/Destino |
|---------|------|-----------|-------|----------------|
| Entrada | NFS  | TCP       | 2049  | sg-EC2         |
| Saída   | NFS  | TCP       | 2049  | sg-EC2         |

#### sg-EC2 (Instâncias EC2)
| Direção | Tipo        | Protocolo | Porta | Origem/Destino |
|---------|------------|-----------|-------|----------------|
| Entrada | HTTP       | TCP       | 80    | sg-ALB         |
| Entrada | MySQL      | TCP       | 3306  | sg-RDS         |
| Saída   | NFS        | TCP       | 2049  | 0.0.0.0/0      |
| Saída   | Todo Tráfego | TCP     | 80    | 0.0.0.0/0      |
| Saída   | MySQL      | TCP       | 3306  | 0.0.0.0/0      |
| Saída   | HTTP       | TCP       | 80    | 0.0.0.0/0      |

---

### 3. Configurar o RDS
- Criar instância MySQL ou MariaDB (qual preferir)  
- Selecionar opção de *Free-tier*  
- Tipo: `db.t3.micro`  
- Associar à VPC criada
- **IMPORTANTE ->** Em configuração adicional escolha o mesmo nome do **Identificador da instância de banco de dados** para evitar possíveis erros
- <img width="1332" height="167" alt="Image" src="https://github.com/user-attachments/assets/fb725fae-a211-40c9-a8e3-0c288a526045" />
- <img width="889" height="166" alt="Image" src="https://github.com/user-attachments/assets/3d0f4120-6df8-4315-95db-8e6547a6a7ef" />

### 4. Configurar o EFS
- Criar sistema de arquivos EFS
- Clicar em **PERSONALIZAR**
- <img width="676" height="684" alt="Image" src="https://github.com/user-attachments/assets/d0b071e2-a5f6-42c6-97f0-483338c207a8" />
- Associar subnets privadas de cada zona, e selecionar o grupo de segurança da EFS
- <img width="1559" height="342" alt="Image" src="https://github.com/user-attachments/assets/1851271f-2a8d-47b9-bc55-4d4ce96b04d4" />
- E criar a EFS

### 5. Criar Launch Template
- Sistema Operacional: Amazon Linux  
- Tipo: `t2.micro`  
- Selecionar VPC do projeto
- Não precisa incluir a sub-rede
- Incluir o Security Group da EC2  
- Incluir script *user-data* para:
  - Instalar WordPress  
  - Montar EFS  
  - Conectar ao RDS  

### 6. Criar Target Group (Grupo de Destino)
- Tipo de destino: **Instâncias**
- Incluir a VPC criada para o projeto
- Selecionar a subnets públicas  
- Caminho de verificação de integridade (Health Check Path): `/` ou `/wp-admin/images/wordpress-logo.svg`
<img width="1073" height="260" alt="Image" src="https://github.com/user-attachments/assets/9e5ff602-7b35-4d4f-a4cc-1d75943d2d28" />

### 7. Configurar Application Load Balancer
- Selecionar o tipo Application Load Balamcer
- Voltado para Internet
- Selecionar a VPC criada para o projeto
- Associar às **subnets públicas**  
- Direcionar tráfego para o **Target Group**  
<img width="1300" height="235" alt="Image" src="https://github.com/user-attachments/assets/a6c0ec88-37fb-4763-b302-2fba238fa973" />

### 8. Criar Auto Scaling Group
- Usar o Launch Template criado
- VPC do projeto
- Configurar nas subnets privadas
- Balanceamento de Carga; Selecione a opção: Anexar a um balanceador de carga existente   
- Associar ao ALB (Application Load Balancer)  

### 9. Resultados Finais
- Acesse a página **Load Balancers** na seção EC2  
- Copie o DNS e abra no navegador  
- Em instantes, aparecerá a tela do WordPress  

### 10. WordPress
<img width="1917" height="918" alt="Image" src="https://github.com/user-attachments/assets/3179a93c-a83b-46e7-820d-b855e306d63d" />


##USER DATA

````bash
#!/bin/bash
set -xe

# ==== VARIÁVEIS DE CONFIGURAÇÃO ====
EFS_ID="fs-xxxxxxxxx" (ID DA SUA EFS)
DB_ROOT_PASSWORD="senhadobd"  (SUA SENHA UTILIZA NA CRIAÇÃO DO BANCO DE DADOS)
DB_NAME="NOME DO BANCO" (O NOME DO BANCO UTILIZADO)
DB_USER="admin" (O USUÁRIO DO BANCO)
DB_PASSWORD="senhadodb" (SUA SENHA UTILIZA NA CRIAÇÃO DO BANCO DE DADOS)

# ==== ATUALIZA SISTEMA E INSTALA DEPENDÊNCIAS ====
yum update -y
yum install -y aws-cli docker amazon-efs-utils

# ==== INICIA DOCKER ====
service docker start
systemctl enable docker
usermod -a -G docker ec2-user

# ==== INSTALA DOCKER COMPOSE ====
curl -SL https://github.com/docker/compose/releases/download/v2.34.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# ==== MONTAGEM DO EFS ====
mkdir -p /mnt/efs
mount -t efs ${EFS_ID}:/ /mnt/efs
echo "${EFS_ID}:/ /mnt/efs efs defaults,_netdev 0 0" >> /etc/fstab

# Permissão para o WordPress escrever no EFS
chown -R 33:33 /mnt/efs

# ==== PASTA DO PROJETO ====
mkdir -p /home/ec2-user/projeto-docker
cd /home/ec2-user/projeto-docker

# ==== ARQUIVO .env ====
cat > .env <<EOL
MYSQL_ROOT_PASSWORD=${DB_ROOT_PASSWORD}
MYSQL_DATABASE=${DB_NAME}
MYSQL_USER=${DB_USER}
MYSQL_PASSWORD=${DB_PASSWORD}
EOL

# ==== ARQUIVO docker-compose.yml ====
cat > docker-compose.yml <<'EOL'
version: '3.8'

services:
  database:
    mem_limit: 2048m
    image: mysql:8.0.43
    restart: unless-stopped
    ports:
      - 3306:3306
    env_file: .env
    environment:
      MYSQL_ROOT_PASSWORD: '${MYSQL_ROOT_PASSWORD}'
      MYSQL_DATABASE: '${MYSQL_DATABASE}'
      MYSQL_USER: '${MYSQL_USER}'
      MYSQL_PASSWORD: '${MYSQL_PASSWORD}'
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - wordpress-network
      
  phpmyadmin:
    depends_on:
      - database
    image: phpmyadmin/phpmyadmin
    restart: unless-stopped
    ports:
      - 8081:80
    env_file: .env
    environment:
      PMA_HOST: database
      MYSQL_ROOT_PASSWORD: '${MYSQL_ROOT_PASSWORD}'
    networks:
      - wordpress-network

  wordpress:
    depends_on:
      - database
    image: wordpress:6.8.2-php8.1-apache
    restart: unless-stopped
    ports:
      - 80:80
    env_file: .env
    environment:
      WORDPRESS_DB_HOST: database
      WORDPRESS_DB_NAME: '${MYSQL_DATABASE}'
      WORDPRESS_DB_USER: '${MYSQL_USER}'
      WORDPRESS_DB_PASSWORD: '${MYSQL_PASSWORD}'
    volumes:
      - /mnt/efs:/var/www/html
    networks:
      - wordpress-network

volumes:
  db-data:

networks:
  wordpress-network:
    driver: bridge
EOL

# ==== SOBE OS CONTAINERS ====
docker-compose up -d
````
