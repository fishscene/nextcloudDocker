# nextcloudDocker
2020/12/18: A quick 'n dirty tutorial for getting nextcloud set up using portainer, docker, and nginx.


# Portainer Stack
- Remove ANY AND ALL TABS and replace with spaces. Always 2 spaces, never 1, or 3: 

      version: '2'
      
      #volumes:
      #  nextcloud:
      #  mariadb:

      services:
        mariadb:
          image: mariadb
          container_name: mariadb
          command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
          restart: unless-stopped
          volumes:
            - mariadb:/var/lib/mysql
          environment:
            - TZ=America/Los_Angeles
            - MYSQL_ROOT_PASSWORD=(changeme)
            - MYSQL_PASSWORD=(changeme2)
            - MYSQL_DATABASE=nextcloud
            - MYSQL_USER=nextcloud

        app:
          image: nextcloud
          container_name: nextcloud
          ports:
            - (port on host):80
          links:
            - mariadb
          volumes:
            - nextcloud:/var/www/html
          environment:
            - TZ=America/Los_Angeles
            - trusted_domains=(ip_address of portainer host, no http/https) (url, no http/https) (url, no http/https etc...)
          restart: unless-stopped

NGINX-MANAGER Web Proxy with SSL
- DETAILS

      -- Scheme: http
      -- Forward Hostname: (IP address of the docker server)
      -- Forward port: (Port of the container)
      -- Websockets Support: Enable
      -- Block Common Exploits: Enable
- CUSTOM LOCATIONS

      -- Location:/.well-known/carddav
      -- Forward Hostname: (IP address of the docker server)/remote.php/dav ## Example: 10.10.10.17/remote.php/dav
      -- Forward Port: (Port of the container)

      -- Location:/.well-known/caldav
      -- Forward Hostname:  (IP address of the docker server)/remote.php/dav ## Example: 10.10.10.17/remote.php/dav
      -- Forward Port: (Port of the container)
- SSL

      -- Force SSL: Enable
      -- HSTS Enable: Enable
      -- HTTP/2 Support: Optional
      -- HSTS Subdomains: Enable


NEXTCLOUD WEBPAGE SETUP
- Storage and Database > select MySQL/MariaDB

      -- Database user --> "nextcloud"
      -- Database password --> password which has been specified in the docker-compose file with MYSQL_ROOT_PASSWORD
      -- Database name --> "nextcloud"
      -- localhost host --> "mariadb"

ALLOW TRUSTED DOMAINS
- Enter in to the nextcloud container console:

      apt-get update
      apt-get install nano
      nano /var/www/html/config/config.php
      
      'trusted_domains' => 
      array (
        0 => '(url, no http/https)',
      ),
  
ENABLE SMB/CIFS in Nextcloud:
- Enter in to the nextcloud container console:

      apt-get update && apt-get install smbclient
- Restart Nextcloud docker container


ENABLE TALK:
- Enter in to the nextcloud container console:

      nano /var/www/html/config/config.php
- Add 2 lines at the bottom before last closing parenthesis:

      'forcessl' => true,
      'overwriteprotocol' => 'https',
- Restart Nextcloud docker container

