# Como instalar e configurar o Nginx para operar como Proxy Reverso

## Introdução

Rodando o Minikube no Multipass, para ter acesso aos serviços fora da maquina virtual foi necessário configurar um **proxy reverso**.

Abaixo estão os passos:

### Instalação

```bash
sudo apt update
sudo apt install nginx
```

### Configuração

O arquivo primário de configuração esta em:
```bash
/etc/nginx/nginx.conf
```

Mas para adicionar novos sites nós vamos trabalhar com os diretórios **site-available** and **site-enabled**.

```bash
/etc/nginx/sites-available/   # Store your site configurations here
/etc/nginx/sites-enabled/     # Enabled configurations are symlinked from sites-available
```

#### Passo a Passo

1. Para evitar erros se for a primeira vez que estiver configurando use o comando abaixo:

```bash
sudo rm /etc/nginx/sites-enabled/*
```

2. Vamos criar um novo arquivo de configuração em sites-available.

```bash
sudo nano /etc/nginx/sites-available/kubernetes-proxy
```

3. Nesse arquivo vamos adicionar a configuração do proxy reverso.

Se estiver utilizando Minikube para descobrir o url do serviço NodePort use o seguinte comando:

```bash
minikube service --url nome_do_servico --namespace=default
```

```nginx
server {
    listen 80;
    server_name _;

    location /myapp {
        proxy_pass http://192.168.49.2:30008/;  # Note the trailing slash
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Add these lines to help with debugging
        proxy_connect_timeout 75s;
        proxy_read_timeout 300s;
        proxy_buffering off;
    }
}
```

4. Confira se o arquivo foi salvo corretamente:

```bash
ls -l /etc/nginx/sites-available/
```

5. Vamos criar um novo link simbolico:

```bash
sudo ln -s /etc/nginx/sites-available/kubernetes-proxy /etc/nginx/sites-enabled/
```

6. Teste a configuração:

```bash
sudo nginx -t
```

7. Reinicie o serviço do Nginx:

```bash
sudo systemctl reload nginx
```

### Acessando fora da máquina virtual

O comando abaixo irá mostrar 3 ips. Considerando o exemplo abaixo:

10.254.65.44 - Este é o IP primário da VM no Multipass, é esse ip que iremos usar para acessar os serviços dentro da VM.
172.17.0.1 - Tipicamente Docker bridge network interface
192.168.49.1 - Este é o Minikube network interface

```bash
multipass list
```
```bash
Name                    State             IPv4             Image
minikube                Running           10.254.65.44     Ubuntu 24.04 LTS
                                          172.17.0.1
                                          192.168.49.1
```

De acordo com a configuração feita acima para acessar o Apache Server dentro da VM vamos usar o seguinte endereço:

[http://10.254.65.44/myapp](http://10.254.65.44/myapp)

## Referências

[how-to-install-nginx-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04)