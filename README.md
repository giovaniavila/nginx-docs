# Configurar Proxy Reverso e Load Balance com Nginx na AWS

## 1. Instalar o Nginx

Se ainda não tiver o Nginx instalado na sua máquina, execute o comando:

```bash
sudo apt update         # Para distribuições baseadas em Debian, como Ubuntu
sudo apt install nginx
```

verificar se o nginx está rodando na sua máquina
```
sudo systemctl status nginx 
```

## 2.  Configurar Hosts Virtuais
entre no arquivo ```cd /etc/nginx/sites-available/default``` e adicione os servidores.

```
# Default server configuration
#
server {
        listen 80;
        listen [::]:80;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
server {
        listen 8080;
        listen [::]:80;

        server_name servidor1;

        root /var/www/servidor1;
        index index.html;

        location / {
                try_files $uri $uri/ =404;
        }
}

server {
        listen 8081;
        server_name servidor2;

        root /var/www/servidor2;
        index index.html;

        location / {
                try_files $uri $uri/ =404;
        }
}

```

## 3. Navegue até o Diretório de Configuração do NGINX
```
cd /etc/nginx/conf.d/
```

## 4. Criar o Arquivo de Configuração do Balanceador de Carga

```
sudo vim /etc/nginx/conf.d/load-balancer.conf
```

## 5. Adicione o Seguinte Conteúdo ao Arquivo

```
# load-balancer.conf

upstream servidorgiovani {
    server localhost:servidor1;
    server localhost:servidor2;
}

server {
    listen 8082;
    server_name load;

    location / {
        proxy_pass http://servidorgiovani; # proxy reverso
    }

    error_page 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}

```

1) Teste a configuração ```sudo nginx -t```
2) reinicie o nginx ```sudo systemctl reload nginx```

## 6 Configuração de Porta no AWS Security Group
Para garantir que o tráfego seja permitido nas portas ```8080```, ```8081``` e ```8082```, adicione regras de entrada no AWS Security Group associado à sua instância EC2:

1) No console da AWS, vá para a página da sua instância EC2.
2) Encontre o Security Group associado à instância.
3) Clique em Editar regras de entrada.
4) Adicione regras para permitir tráfego HTTP ```(8080, 8081,8082)```, como no exemplo abaixo:
- <strong>Tipo:</strong> HTTP
- <strong>Protocolo:</strong> TCP
- <strong>Porta:</strong> 80
- <strong>Fonte/origem:</strong>  0.0.0.0/0 (para permitir de qualquer IP)

## 7. Agora você pode testar através de url http com o ip da sua máquina Nginx
Acesse pela Url do seu navegador

```
http://<ip da sua máquina em que está configurado o Nginx e o nome da upstream configurada em load-balancer.conf>
```
- exemplo: ```http://<ip publico da sua maquina nginx>:8082```

