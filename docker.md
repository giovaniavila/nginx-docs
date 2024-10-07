# Adicionando docker as VM'S

## 1) Adicionar docker na Vm server
- Adicione o docker na máquina em que os containers irão rodar

```
sudo apt update
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

## 2) Criar o container dos servidores 
- crie os dockerfiles e os arquivos index.html para cada máquina server
  ``` mkdir servidor1 servidor2 servidor3 ```
  exemplo:
  
  ```
   cd servidor1 vim Dockerfile
   ```
  - Conteúdo do dockerfile:
    ```
    FROM nginx:alpine
    COPY ./index.html /usr/share/nginx/html/index.html
    ```
  - Crie o arquvo index.html para cada servidor:
    
    ```
    vim index.html
    ```
    
    ```
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Servidor 1</title>
    </head>
    <body>
      <h1>Bem-vindo ao Servidor 1</h1>
    </body>
    </html>
    ```

## 3) Agora crie os comandos de build: 
```
docker build -t servidor1 ./servidor1
docker build -t servidor2 ./servidor2
docker build -t servidor3 ./servidor3
```

## 4) Rode os containers
```
docker run -d --name servidor1 -p 8080:80 servidor1
docker run -d --name servidor2 -p 8081:80 servidor2
docker run -d --name servidor3 -p 8082:80 servidor3
```

## 5) Na máquina nginx:
- Altere a configuração de balanceamento em conf.d
  ```
  upstream servidorgiovani {
    server <IP-VM-SERVER>:8080;  # Porta do container Servidor 1
    server <IP-VM-SERVER>:8081;  # Porta do container Servidor 2
    server <IP-VM-SERVER>:8082;  # Porta do container Servidor 3
  }

  server {
    listen 8083;
    server_name load;

    location / {
        proxy_pass http://servidorgiovani;
    }
  }
  ```
  Reinicie o nginx ```sudo systemctl reload nginx```

## 6) Libere as portas necessárias no grupo de segurança:
Para garantir que o tráfego seja permitido nas portas ```8080```, ```8081```, ```8082``` e ```8083```, adicione regras de entrada no AWS Security Group associado à sua instância EC2:

1) No console da AWS, vá para a página da sua instância EC2.
2) Encontre o Security Group associado à instância.
3) Clique em Editar regras de entrada.
4) Adicione regras para permitir tráfego HTTP ```(8080, 8081,8082, 8083)```, como no exemplo abaixo:
- <strong>Tipo:</strong> HTTP
- <strong>Protocolo:</strong> TCP
- <strong>Porta:</strong> 80
- <strong>Fonte/origem:</strong>  0.0.0.0/0 (para permitir de qualquer IP)
