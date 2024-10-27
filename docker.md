# Adicionando docker as VM'S

## 1) Adicionar docker na Vm server
- Adicione o docker na máquina em que os containers irão rodar

```
sudo apt update
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

- Instale o docker compose
  ``` sudo apt install docker-compose ```

## 2) Criar o container dos servidores 
- crie os arquivos para cada máquina server
  ``` mkdir servidor1 servidor2 servidor3 ```

- Adicione os arquivos html nos servidores
```
mkdir -p servidor1 servidor2 servidor3 && echo '<html><body><h1>Servidor 1</h1></body></html>' > servidor1/index.html && echo '<html><body><h1>Servidor 2</h1></body></html>' > servidor2/index.html && echo '<html><body><h1>Servidor 3</h1></body></html>' > servidor3/index.html
```

- crie o arquivo docker-compose.yml e adicione:
```
  version: '3'
services:
  servidor1:
    build:
      context: ./servidor1
    ports:
      - "8080:80"
  servidor2:
    build:
      context: ./servidor2
    ports:
      - "8081:80"
  servidor3:
    build:
      context: ./servidor3
    ports:
      - "8082:80"
```

- Estrutura de como ficou os arquivos <br>
ubuntu@ip-172-33-81-72:~$ ls <br>
docker-compose.yml  server1  server2  server3


## 3) Executando os containers: 
- Para executar:
``` docker-compose up -d ```

- verifique o funcionamento dos servidores
```
  curl http://localhost:8080  # Deve retornar "Servidor 1"
  curl http://localhost:8081  # Deve retornar "Servidor 2"
  curl http://localhost:8082  # Deve retornar "Servidor 3"
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
    server_name _;

    location / {
        proxy_pass http://servidorgiovani;
        proxy_cache off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        add_header Cache-Control no-store; #importante para que funcione no reload da página
    }

    error_page 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
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
