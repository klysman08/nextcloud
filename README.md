# nextcloud
Nextcloud é uma suíte de software cliente-servidor para criar e usar serviços de hospedagem de arquivos. Ele fornece funcionalidade semelhante ao Dropbox, Office 365 ou Google Drive quando usado com suítes de escritório integradas.


# Instalação do docker e plugins

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo usermod -aG docker $USER
sudo systemctl restart docker
#Após executar esses comandos, faça logout e faça login novamente na sua sessão para que as alterações tenham efeito. Depois disso, você poderá executar comandos Docker sem a necessidade de sudo.
```

# Alocando memoria SWAP

Esse processo melhora o desempenho da maquina virtual para o nextcloud

```bash
sudo swapon --show
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

# Portainer:

```bash
sudo docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer:/data portainer/portainer-ce:latest
ou
sudo docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer:/data portainer/portainer-ce:latest -H unix:///var/run/docker.sock
```

[http://SEUIP:9000/](http://SEUIP:9000/)

```bash
crie um user
```

# Portas:

Permita que todas as portas possam acessar a porta 9000 da máquina

Exemplo na oracle cloud:

![Untitled](Next%20Cloud%2088222f79c47345c4a2676661991e82f4/Untitled.png)

# Stack Nextcloud

Docker compose:

```bash
version: "2"
services:
  app:
    depends_on:
      - db
    environment:
      - MYSQL_PASSWORD=SENHA
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_HOST=db
    image: nextcloud
    links:
      - db
    ports:
      - "8080:80"
    restart: always
    volumes:
      - "/mnt/docker/nextcloud/nextcloud:/var/www/html"
      - "/mnt/docker/nextcloud/apps:/var/www/html/custom_apps"
      - "/mnt/docker/nextcloud/config:/var/www/html/config"
      - "/mnt/docker/nextcloud/data:/var/www/html/data"
      - "/mnt/docker/nextcloud/theme:/var/www/html/themes/<YOUR_CUSTOM_THEME>"
  db:
    command: "--transaction-isolation=READ-COMMITTED --binlog-format=ROW"
    environment:
      - MYSQL_ROOT_PASSWORD=SENHA
      - MYSQL_PASSWORD=SENHA
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
    image: "mariadb:10.6"
    restart: always
    volumes:
      - "/mnt/docker/nextcloud/db:/var/lib/mysql"
```

# CloudFlare

1. Crie o acesso ao Zero Trust
2. Crie um novo Tunnel
3. Cria a imagem em docker:
    
    > docker run cloudflare/cloudflared:latest tunnel --no-autoupdate run --token CHAVE...
    > 

> Dê acesso do **nextcloud-app-1** a bridge que foi criada
> 
> 
> Join network
> 

****Route Traffic for nextcloud****

1. subdomain: nextcloud
2. domain: krmdigital.com
3. type: http
4. url: IP-DA-BRIDGE:80

[Nextcloud](http://nextcloud.seudominio.com/)

