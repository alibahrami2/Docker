This Setup has 2 way for SSL Termination, First way is self-signed SSL certificate, the second way is a certbot certificate, choose what u want. 

0. Generate the self-signed SSL certificate:
```bash
mkdir -p /path/to/your/certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /path/to/your/certs/selfsigned.key -out /path/to/your/certs/selfsigned.crt
```

Replace `/path/to/your/certs` with the `actual directory` where you want to store your certificate and key files. During this process, you will be asked for details such as country, state, organization, and Common Name (use your domain or server IP address).

********** If you want to use this on your organization, i suggest using this steps: **********
1. Create the Docker Compose file (`docker-compose.yml`).

2. Create the Nginx configuration file (`nginx.conf`).

3. Initial Certificate Request:
Before you start the Docker containers, you need to manually request the certificates from Let's Encrypt. You can do this by running the Certbot container with the following command:

```bash
docker run -it --rm --name certbot \
    -v /path/to/your/letsencrypt:/etc/letsencrypt \
    -v /path/to/your/nginx/conf:/var/www/certbot \
    certbot/certbot certonly --webroot -w /var/www/certbot -d yourdomain.com
```

Make sure to replace /path/to/your/letsencrypt and /path/to/your/nginx/conf with the actual paths.

4. Start the Docker Containers:
After you have obtained the certificates, you can start the containers using Docker Compose:

```bash
docker-compose up -d
```
This setup includes:

`Nextcloud` for your main application.
`MySQL` as the database service.
`Nginx` as a reverse proxy with SSL termination.
`Certbot` to automatically renew the Let's Encrypt certificates.
Make sure to replace yourdomain.com with your actual domain name and adjust any paths as needed for your environment. 