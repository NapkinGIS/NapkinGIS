## Server deployment with Let's Encrypt certificate

Set of commands to simplify the setup of new deployments.


Before first start, adjust the [configuration](#startup-configuration)

Temporary switch to configuration with http protocol only
```
rm nginx/conf.letsencrypt/default.conf
ln -s conf.http nginx/conf.letsencrypt/default.conf
```

Start NapkinGIS
```
docker-compose up -d
```

Create Let's Encrypt certificate
```
docker-compose -f certbot.yml run --rm certbot certonly --agree-tos -a webroot \
    --webroot-path /var/www/certbot/ \
    --email contact@napkingis.no \
    -d web.napkingis.no
```

Switch to normal configuration and restart nginx server
```
rm nginx/conf.letsencrypt/default.conf
ln -s conf.letsencrypt nginx/conf.letsencrypt/default.conf
docker-compose restart nginx
```

NapkinGIS should now be working over https. Before starting to use NapkinGIS, you will have to [initialize the DB](#db-initialization).

Renew certificate before it will expire:
```
docker-compose -f certbot.yml run --rm certbot renew
docker-compose kill -s HUP nginx
```

Tip: use `--dry-run` option in certbot commands for testing


## Startup configuration

### Django configuration

You can extend default Django configuration in ```django/settings.custom/``` directory.


When using the optional *accounts* extension (```GISQUICK_ACCOUNTS_ENABLED=True```), you need to set up correct email configurations:
- Check/adjust ```<settings-directory>/email.py```
- Set variables ```DJANGO_EMAIL_HOST_USER``` and ```DJANGO_EMAIL_HOST_PASSWORD``` in ```django.env``` (or in ```<settings-directory>/email.py```) 


## DB initialization

Create database (django service must be running)
```
docker-compose exec django django-admin makemigrations app
docker-compose exec django django-admin migrate
```

Create superuser account
```
docker-compose exec django django-admin createsuperuser
```

Create regular users from admin interface running on <server-url>/admin


## Useful commands

Generate new secret key
```
tr -dc 'a-z0-9!@#%^&*(-_=+)' < /dev/urandom | head -c50
```

Reload NGINX server
```
docker-compose kill -s HUP nginx
```

Update web app
```
docker-compose up -d web-map
```
Note: use ```--force-recreate``` flag to update assets even if image didn't change

## Notes

### Define upload & project limits

Upload file size limit is defined in `docker-compose.yml`:

```
MAX_FILE_UPLOAD=50M
```

Project size limit is defined by two files: `docker-compose.yml`

```
MAX_PROJECT_SIZE=150M
```

and `nginx/conf.letsencrypt/locations`

```
client_max_body_size 150M;
```
