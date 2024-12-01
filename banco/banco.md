# Adicionando o banco com os containers

## 1) Criar uma instância RDS MySQL
 1.1. Criar uma instância RDS MySQL
- Acesse o Console da AWS RDS:
  <br>
2) Vá para o console RDS.
- Clique em "Create database":
- Escolha Standard create.
- Engine: MySQL.
  <br>
3) Configurações do Banco:
- Versão do MySQL: Escolha a versão mais recente suportada.
- DB instance identifier: Nome da instância, como mydatabase.
- Master username: Defina o nome de usuário, ex.: admin.
- Password: Configure uma senha forte.
  <br>
4) Configurações de rede:
- Certifique-se de que a instância está em uma VPC que permite comunicação com sua VM (server).
- Configure o Security Group para permitir conexões vindas do IP da sua VM Docker.
- OBS: caso tenha problemas, coloque public access
  <br>
5) Storage:
- Escolha um tamanho inicial, como 20 GB.
- Finalize a criação clicando em Create database.
  <br>
6) Configurar o Security Group
- Vá até o EC2 Dashboard e acesse Security Groups.
- Adicione uma regra de entrada (Inbound Rule):
- Type: MySQL/Aurora.
- Port Range: 3306.
- Source: O IP da sua VM ou 0.0.0.0/0 (não recomendado para produção).

## 2) Criar o banco de dados e tabelas no mysql
- Após a configuração do RDS, conecte-se a ele para criar o banco e tabelas:

### na sua vm:
```
mysql -h <RDS_ENDPOINT> -u admin -p
```

- Substitua <RDS_ENDPOINT> pelo endpoint fornecido no console RDS.
- Digite a senha configurada.

### 2.1 criar o banco de e tabelas:


```
CREATE DATABASE project_data;

USE project_data;

CREATE TABLE server_data (
    id INT AUTO_INCREMENT PRIMARY KEY,
    server_name VARCHAR(255) NOT NULL,
    data TEXT NOT NULL
);

INSERT INTO server_data (server_name, data) VALUES 
('server1', 'Dados do Server 1'),
('server2', 'Dados do Server 2'),
('server3', 'Dados do Server 3');
```

## 3 Configurar os Servidores Docker para Conectar ao RDS
- Cada servidor no Docker precisa estar configurado para se conectar ao banco de dados RDS.
- Deve-se atualizar o docker compose:
  
```
version: '3.8'

services:
  server1:
    image: nginx
    environment:
      DB_HOST: <RDS_ENDPOINT>
      DB_USER: admin
      DB_PASSWORD: <YOUR_PASSWORD>
      DB_NAME: project_data
      SERVER_NAME: server1

  server2:
    image: nginx
    environment:
      DB_HOST: <RDS_ENDPOINT>
      DB_USER: admin
      DB_PASSWORD: <YOUR_PASSWORD>
      DB_NAME: project_data
      SERVER_NAME: server2

  server3:
    image: nginx
    environment:
      DB_HOST: <RDS_ENDPOINT>
      DB_USER: admin
      DB_PASSWORD: <YOUR_PASSWORD>
      DB_NAME: project_data
      SERVER_NAME: server3
```

- Substitua <RDS_ENDPOINT> e <YOUR_PASSWORD> pelos valores configurados.

## 4 Rodar e Testar os Servidores
```
 docker-compose up -d
```

## 5 teste a conexão com o banco
1) Execute o para definir as variaveis de ambiente
```
export DB_HOST=<endpoint da database>
export DB_PORT=3306 (para mysql)
export DB_USER=admin (nome do user que voce definiu)
export DB_PASSWORD=<senha do seu banco>
export DB_NAME=<nome do banco que voce criou dentro do host>
```
- Após pressionar enter, você deve ser capaz de entrar no banco de dados.

2) verifique as variáveis, se necessario:
```
echo $DB_HOST
echo $DB_PORT
echo $DB_USER
echo $DB_PASSWORD
echo $DB_NAME
```
