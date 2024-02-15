# Create a docker private registry

## On EC2 instance-1:

Edit daemon.json to configure our private registry
```
sudo nano /etc/docker/daemon.json
```
Add the below line in your `/etc/docker/daemon.json` 
```
{
    "insecure-registries": ["Public IPv4 DNS"]
}
```
*Public IPv4 DNS is the domain name given to every EC2 Instances*

And restart your docker
```
sudo systemctl restart docker
```
### Adding certifications to our private registry

Create self signed TLS certificate and TLS key
```
openssl req -newkey rsa:4096 -nodes -sha256 -keyout /home/ubuntu/localhub/localhub.key -x509 -days 365 -out /home/ubuntu/localhub/localhub.crt
```

*openssl req: This command is used for certificate requests and certificate generating utility*

*-newkey rsa:4096: Generate a new RSA private key with a length of 4096 bits*

*-nodes: Do not encrypt the private key*

*-sha256: Use the SHA-256 hash function for the certificate request*

*-x509: Output a self-signed certificate*

*-days 365: Set the validity period of the certificate to 365 days*

### Create a user name and password for your docker registry
```
docker run --rm httpd:alpine htpasswd -Bbn user pass > htpasswd
```

*--rm: Removes the container automatically after it exits, making it a temporary or disposable container*

*httpd:alpine: Specifies the Docker image to use, in this case, httpd:alpine. This image contains the Apache HTTP Server and is based on the Alpine Linux distribution*

*htpasswd: This is a command-line utility for managing user files for basic authentication, commonly used with Apache HTTP Server*

*-B: Specifies the use of the bcrypt password algorithm. Bcrypt is a more secure hashing algorithm compared to the default crypt algorithm*

*-bn: Indicates that we are providing the password on the command line, rather than prompting for it interactively*

### Run the registry container
```
sudo docker run -d -p 443:443 --name local.hub -v /home/ubuntu/localhub/certs:/certs -v /home/ubuntu/localhub/registry:/var/lib/registry -v /home/ubuntu/localhub/auth:/auth -e REGISTRY_HTTP_ADDR=0.0.0.0:443 -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/localhub.crt -e REGISTRY_HTTP_TLS_KEY=/certs/localhub.key registry
```

*-e "REGISTRY_AUTH=htpasswd": Sets the authentication method to "htpasswd"*

*-e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm": Specifies the realm for HTTP Basic Authentication*

*-e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd: Sets the path to the htpasswd file*

*-e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/localhub.crt: Set the environment variable REGISTRY_HTTP_TLS_CERTIFICATE to specify the path to the TLS certificate file inside the container*

*-e REGISTRY_HTTP_TLS_KEY=/certs/localhub.key: Set the environment variable REGISTRY_HTTP_TLS_KEY to specify the path to the TLS private key file inside the container*

### Check weather your docker private registry is working or not

Pull any desired docker image from docker hub.
Here I am pulling nginx
```
docker pull nginx
```
Tag the image with name and version
```
docker tag iamge_id your_regsitry/desired_name:version
```
Push the image to your private registry
```
docker push your_registry/given_name:version
```
## On EC2 instance-2:

AS Docker, by default, considers registries with self-signed certificates to be insecure we need to edit `daemon.json` to configure our private registry
```
sudo nano /etc/docker/daemon.json
```
Add the below line in your `/etc/docker/daemon.json`
```
{
    "insecure-registries": ["Public IPv4 DNS"]
}
```
*Here you should provide the Public IPv4 DNS of EC2 instance-1*

And you can pull the previous image which you pushed to your docker repository
```
docker pull your_registry/given_name:version
```