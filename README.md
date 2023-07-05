# Nextcloud

O Nextcloud é uma suíte de software cliente-servidor para criar e utilizar serviços de hospedagem de arquivos. Ele oferece funcionalidades semelhantes às do Dropbox, Office 365 ou Google Drive, quando utilizado com suítes de escritório integradas.

## Instalação do Docker e plugins

Execute os seguintes comandos para instalar o Docker e os plugins necessários:

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo usermod -aG docker $USER
sudo systemctl restart docker
```

Após executar esses comandos, faça logout e faça login novamente na sua sessão para que as alterações tenham efeito. Depois disso, você poderá executar comandos Docker sem a necessidade de sudo.

## Alocando memória SWAP

Esse processo melhora o desempenho da máquina virtual para o Nextcloud.

Execute os seguintes comandos:

```bash
sudo swapon --show
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

## Portainer

O Portainer é uma interface de gerenciamento do Docker que permite administrar facilmente os seus containers. Execute o seguinte comando para executar o Portainer:

```bash
sudo docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer:/data portainer/portainer-ce:latest
ou
sudo docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer:/data portainer/portainer-ce:latest -H unix:///var/run/docker.sock
```

Acesse o Portainer em [http://SEUIP:9000/](http://SEUIP:9000/) e crie um usuário.

## Portas

Permita que todas as portas possam acessar a porta 9000 da máquina. Por exemplo, na Oracle Cloud, você pode configurar isso através da interface do Oracle Cloud Console.

## Stack Nextcloud

Utilize o Docker Compose abaixo para configurar o Nextcloud:

```yaml
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
      - "/mnt/docker/nextcloud/apps:/var/www/html"
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

## CloudFlare

Aqui estão os passos para configurar o CloudFlare:

1. Crie o acesso ao Zero Trust.
2. Crie um novo túnel.
3. Crie a imagem em Docker:

   ```
   docker run cloudflare/cloudflared:latest tunnel --no-autoupdate run --token CHAVE...
   ```

4. Dê acesso do **nextcloud-app-1** à bridge que foi criada.
5. Junte-se à rede.

## Roteamento de tráfego para o Nextcloud

Configure o roteamento de tráfego para o Nextcloud da seguinte forma:

1. Subdomínio: nextcloud.
2. Domínio: krmdigital.com.
3. Tipo: HTTP.
4. URL: IP-DA-BRIDGE:80.

Acesse o Nextcloud em http://nextcloud.seudominio.com/