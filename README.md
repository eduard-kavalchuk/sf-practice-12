### Create a corporate Certification Authority
1. Create an Ubuntu VM in Yandex Cloud (server VM).
2. Install an Nginx server.
3. Use Yandex Cloud DNS to create an **internal** domain zone and add a new type-A record that associates a cloud-internal ```sf-practice.com``` domain name with a public IP address of the VM.
4. Create a private key and a self-signed certificate:
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout root-key.key -out root-crt.crt
```
When answering questions it is a must to set a ```Common Name``` as ```sf-practice.com```. Otherwise, the sertificate will not be accepted when connecting to the server.  
5. Copy ```root-key.key``` and ```root-crt.crt``` to ```/etc/nginx/ssl``` directory.  
6. Make ```server``` blocks in ```/etc/nginx/sites-enabled/default``` config file look like the following:
```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name <vm-public-ip>;
}

server {
    listen 443 ssl http2 default_server;
    listen [::]:443 ssl http2 default_server;
    ssl_certificate 	/etc/nginx/ssl/root-crt.crt;
    ssl_certificate_key /etc/nginx/ssl/root-key.key;
}
```
7. Check integrity of the resulting configuration:
```
sudo nginx -t
```
8. Reload Nginx:
```
sudo systemctl restart nginx
```
Use your browser to access the server via a secure protocol:
```
https://<vm-public-ip>
```
It will complain that the connection is not secure. Accept warnings and open the web page. Make sure that information about a certificate reported on the page is correct.    
9. Create another Ubuntu VM.  
10. Copy the root certificate ```root-crt.crt``` from the server VM.  
11. Install the certificate:
```
sudo apt-get install -y ca-certificates
sudo cp root-crt.crt /usr/local/share/ca-certificates
sudo update-ca-certificates
```
12. Attempt to connect to ```sf-practice.com``` via http and https:
```
curl http://sf-practice.com | head -10
curl https://sf-practice.com | head -10
```
Make sure the web page is fetched and there are no complains regarding certificates in case of the https connection.
