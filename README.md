# 3º Trabalho - Estágio Compass UOL - AWS & DevSecOps

Migração de um e-commerce no modelo on-premisse (local) para a nuvem AWS

## Descrição do ambiente on-premisse:
- 1 servidor para banco de dados Mysql
- 1 servidor para a aplicação usando React
- 1 servidor de web server e que armazena estáticos como fotos e links

## Descrição nova arquitetura:
![diagrama](https://github.com/MuriloScheunemann/Compass3-Migracao-AWS/assets/122695407/eb94ac2d-1383-47e4-b6f4-d333e3524148)
### Geral: 

Os domínios de acesso à aplicação, para usuários e para o administrador, são configurados no Rota 53, o serviço de DNS da AWS. O tráfego de entrada passa por um WAF – Web Application Firewall, que é uma camada a mais de segurança para a aplicação. O tráfego é direcionado para um ALB – Application Load Balancer, que o distribui entre as máquinas do cluster Kubernetes. O cluster Kubernetes é implementado com o serviço de EKS – Elastic Kubernetes Service, o qual gerencia o cluster e contém um AutoScaling para as máquinas. As máquinas do cluster rodam a aplicação. Elas se conectam ao banco de dados da aplicação, que é uma instância Amazon RDS com engine Aurora. Os snapshots de backup do RDS Aurora são armazenados num Bucket S3 Glacier Instant Retrieval. Para comunicação externa, as máquinas do cluster contam com o NAT Gateway, o elemento que possibilita conexão externa para elas. O ambiente ainda oferece um EFS – Elastic File System, o serviço de NFS da Amazon, que é montado nas máquinas armazenando arquivos compartilhados da aplicação. Os estáticos do site são armazenados num Bucket S3 Standard. A parte de monitoramento é feita com o Amazon CloudWatch, para análise de métricas e logs, em conjunto com o Amazon SNS, para envio de notificações. Os usuários do ambiente AWS são configurados no AWS IAM.
### Camada de entrada: 
- A resolução de DNS do site é feita com o Rota 53: os domínios de usuário e de administrador;
- O tráfego é filtrado pelo WAF (Web Application Firewall);
- O ALB (Application Load Balancer) distribui o tráfego entre os nodes do cluster Kubernetes;
### Camada de aplicação: 
- O cluster Kubernetes é implementado com o EKS (Elastic Kubernetes Service), que gerencia autonomamente os nodes;
- Os nodes do cluster são instâncias EC2 do tipo m6g.medium;
- O cluster EKS contém um AutoScaling, serviço que garante dimensionamento horizontal flexível. Inicialmente, é definida a quantidade de 2 máquinas (1 em cada AZ);
### Camada de dados: 
- O Amazon RDS Aurora é o banco de dados da aplicação, totalmente compatível com Mysql. Ele possui modo Multi-AZ: cria uma réplica de si mesmo em outra Zona de Disponibilidade;
- O Amazon S3 é usado para armazenamento de estáticos do site e de snapshots de backup do RDS;
- Um EFS (Elastic File System) é utilizado para armazenamento de arquivos compartilhados da aplicação;
### Acesso administrativo: 
- O ambiente interno da aplicação pode ser acessado através de um bastion host, que é uma instância EC2, tipo t2.micro. Este bastion pode ser usado para fins de acesso ao banco de dados, aos arquivos compartilhados (EFS), acesso aos nodes...
- Existe um AutoScaling Group configurado para a instância Bastion, definido para as duas Zonas de Disponibilidade, garantindo alta disponibilidade de acesso administrativo;
- São definidas credenciais de acesso temporário, roles (funções de permissionamento) e demais mecanismos de segurança com os serviços de IAM (Identity Access Managment);
### Configuração de rede: 
- A aplicação é hospedada na região AWS “sa-east-1” (São Paulo) e divida entre duas Zonas de Disponibilidade, sa-east-1a e sa-east-1b, garantindo alta disponibilidade;
- A rede interna da aplicação é implementada com uma VPC (Virtual Private Network), abrangendo as duas Zonas de Disponibilidade (sa-east-1a e sa-east-1b);
- Dentro da VPC, em cada uma das Zonas de Disponibilidade, existe:
  - uma subnet pública, com rotas para um Internet Gateway. Ela contém o Bastion Host, o Nat gateway e Load Balancer;
  - uma subnet privada, contendo os nodes do cluster EKS;
  - uma subnet privada, contendo o banco de dados RDS Aurora;
- O esquema de subnets é espelhado entre as duas AZs. Trata-se portanto de 6 subnets ao todo;
- Load Balancer, Worker Nodes e Bastion Host possuem Security Group associado;
## Especificações técnicas
Instâncias EC2 Worker Nodes:
  - m6g.medium (1 vCPU; 4GB de memória RAM)
  - 20GB de volume EBS

Instância EC2 Bastion Host:
  - t2.micro (1 vCPU; 1GB de memória RAM)

Banco de dados RDS Aurora:
  - Aurora Standard
  - db.t4g.medium (2 vCPU; 4GB de memória RAM;)
  - 300GB de armazenamento
  - 300GB de backup
  - 1000GB de exportação de snapshots por mês

## Estimativa de Custo
![custo](https://github.com/MuriloScheunemann/Compass3-Migracao-AWS/assets/122695407/b94df13c-34dd-4d85-ab36-d9f1b3843fa8)






