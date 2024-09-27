# nginx-docs

# Configurar Proxy Reverso com Nginx na AWS

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

## 2. Configurar o proxy reverso

Edite o arquivo de configuração do nginx em ```/etc/nginx/sites-available/defaul```:
```
sudo vim /etc/nginx/sites-available/default
```

Adicione ou modifique o seguinte bloco de configuração (substitua conforme necessário):

```

server {
    listen 80;
    server_name your_domain_or_public_ip;

    location / {
        proxy_pass http://127.0.0.1:3000;  # O serviço rodando na sua máquina
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

- altere ```your_domain_or_public_ip```  para o domínio que deseja usar ou o IP público da instância.
- O ```proxy_pass```  está redirecionando as requisições para o serviço local rodando na porta ```3000```

## 3. Testar a configuração do nginx

Antes de reiniciar o Nginx, verifique se a configuração está correta:

```
sudo nginx -t
```

Reinicie o Nginx para aplicar as mudanças:
```
sudo systemctl restart nginx
```

## 4.  Configurar o Security Group (AWS)

Verifique se a porta ```80``` (HTTP) ou ```443``` (HTTPS) está aberta no Security Group da sua instância EC2.

1) No console da AWS, vá para a página da sua instância EC2.
2) Encontre o Security Group associado à instância.
3) Clique em Editar regras de entrada.
4) Adicione regras para permitir tráfego HTTP ```(80)``` ou HTTPS ```(443)```, como no exemplo abaixo:
- <strong>Tipo:</strong> HTTP
- <strong>Protocolo:</strong> TCP
- <strong>Porta:</strong> 80
- <strong>Fonte/origem:</strong>  0.0.0.0/0 (para permitir de qualquer IP)
