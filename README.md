☁️ Arquitetura do Projeto

A infraestrutura foi projetada para ser resiliente e escalável:

Auto Scaling Group (ASG) com múltiplas instâncias EC2 executando WordPress.

Application Load Balancer (ALB) para balancear o tráfego de entrada.

Amazon EFS para armazenamento compartilhado de arquivos do WordPress (mídias, plugins etc.).

Amazon RDS (MySQL/MariaDB) para armazenar dados do site.

🛠️ Componentes na AWS
🔹 VPC Personalizada

Subnets: 2 públicas e 4 privadas, distribuídas em duas Zonas de Disponibilidade (AZs).

Internet Gateway (IGW): Permite acesso à internet pelas subnets públicas.

NAT Gateway: Localizado nas subnets públicas, permitindo que instâncias privadas acessem a internet.

🔐 Security Groups
sg-ALB (Application Load Balancer)
Direção	Tipo	Protocolo	Porta	Origem/Destino
Entrada	HTTP	TCP	80	0.0.0.0/0
Saída	HTTP	TCP	80	sg-EC2
sg-RDS (Banco de Dados MySQL/MariaDB)
Direção	Tipo	Protocolo	Porta	Origem/Destino
Entrada	MySQL/Aurora	TCP	3306	sg-EC2
Saída	MySQL/Aurora	TCP	3306	sg-EC2
sg-EFS (Armazenamento EFS)
Direção	Tipo	Protocolo	Porta	Origem/Destino
Entrada	NFS	TCP	2049	sg-EC2
Saída	NFS	TCP	2049	sg-EC2
sg-EC2 (Instâncias EC2)
Direção	Tipo	Protocolo	Porta	Origem/Destino
Entrada	HTTP	TCP	80	sg-ALB
Entrada	MySQL	TCP	3306	sg-RDS
Saída	NFS	TCP	2049	0.0.0.0/0
Saída	Todo Tráfego	TCP	80	0.0.0.0/0
Saída	MySQL	TCP	3306	0.0.0.0/0
Saída	HTTP	TCP	80	0.0.0.0/0
🚀 Etapas de Implementação

Criar a VPC

2 subnets públicas

4 subnets privadas

IGW e NAT Gateway

Criar Security Groups (conforme tabelas acima)

Configurar o RDS

Criar instância MySQL ou MariaDB (Free-tier: db.t3.micro)

Associar à VPC do projeto

Configurar o EFS

Criar sistema de arquivos NFS

Associar subnets privadas

Criar Launch Template

Sistema Operacional: Amazon Linux

Tipo: t2.micro

Script user-data para:

Instalar WordPress

Montar EFS

Conectar ao RDS

Criar Target Group

Tipo de destino: Instâncias

Health Check Path: / ou /wp-admin/images/wordpress-logo.svg

Configurar Application Load Balancer

Associar às subnets públicas

Direcionar tráfego para o Target Group

Criar Auto Scaling Group

Usar Launch Template criado

Associar ao ALB

Configurar nas subnets privadas

Resultados Finais

Acesse Load Balancers na seção EC2

Copie o DNS e abra no navegador para ver a tela do WordPress

🖼️ Visualização do WordPress

⚠️ Atenção: para exibir a imagem corretamente no README do GitHub, coloque o arquivo dentro da pasta do projeto (ex: images/) e use caminho relativo:

![WordPress](images/image.png)


Se quiser manter o mesmo nome de arquivo (image.png), crie uma pasta images e mova a imagem para lá.

⚠️ Cuidados para Contas de Estudo

Monitore custos diariamente no Cost Explorer

Remova recursos ao finalizar (VPCs, EC2, RDS etc.)

EC2:

Free-tier compatível

Adicionar tags: Name, CostCenter, Project

RDS:

Tipo: db.t3g.micro

Sem Multi-AZ
