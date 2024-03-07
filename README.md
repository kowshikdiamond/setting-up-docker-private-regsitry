# Setting Up a Private Docker Registry

**Introduction:**

This guide will walk you through the process of setting up a private Docker registry. The registry will be secured with TLS certificates and basic authentication, providing a secure environment for storing and retrieving container images.

**Note:** Feel free to adapt this guide for your local environment, following the same principles outlined here.

**Prerequisites:**

To embark on this journey, ensure that you have:
- Two AWS EC2 instances
- Docker installed on your instances

Now, let's dive into the step-by-step process of creating your secure Docker registry.

**Step 1:** Configure Docker Daemon

Edit the Docker daemon configuration file to allow communication with your private registry. Open the configuration file:
```
sudo nano /etc/docker/daemon.json
```
Add the following line, replacing "Public IPv4 DNS" with the domain name of your Instance-1:
```
{
    "insecure-registries": ["Public IPv4 DNS"]
}
```
Restart Docker to apply the changes:
```
sudo systemctl restart docker
```
**Step 2:** Generate TLS Certificates

Create self-signed TLS certificates and keys for securing the registry. Run the following command:
```
openssl req -x509 -newkey rsa:4096 -keyout registry.key -out registry.crt -days 365 -nodes -subj "/CN=my-registry.local" -extensions v3_req -config <(cat /etc/ssl/openssl.cnf <(printf "[v3_req]\nsubjectAltName=DNS:local-docker-registry,DNS:localhost"))
```
**Note:** Replace local-docker-registry with your desried name


**Step 3:** Create User Credentials

Generate a username and password for Docker registry authentication using the htpasswd utility:
```
docker run --rm httpd:alpine htpasswd -Bbn user pass > /home/ubuntu/localhub/auth/htpasswd
```

**Step 4:** Run Docker Registry Container 

Start the Docker registry container with TLS and basic authentication:
```
sudo docker run -d -p 443:443 --name local.hub \
-v /home/ubuntu/localhub/certs:/certs \
-v /home/ubuntu/localhub/registry:/var/lib/registry \
-v /home/ubuntu/localhub/auth:/auth \
-e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
-e "REGISTRY_AUTH=htpasswd" \
-e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
-e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
-e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/localhub.crt \
-e REGISTRY_HTTP_TLS_KEY=/certs/localhub.key \
registry
```
**Step 5:** Check Docker Private Registry

Pull an image from Docker Hub, tag it, and push it to your private registry:
```
docker pull nginx
````
```
docker tag nginx Public_IPv4_DNS/nginx:latest
```
```
docker push Public_IPv4_DNS/nginx:latest
```
## Configure Docker Daemon on another instance:

**Step 6:** Edit the Docker daemon configuration file:
```
sudo nano /etc/docker/daemon.json
```
Add the following line, replacing "Public IPv4 DNS" with the domain name of Instance-1:
```
{
    "insecure-registries": ["Public IPv4 DNS"]
}
```
**Step 7:** Pull Image from Private Registry on Instance-2

Pull the image from the private registry hosted on Instance-1:
```
docker pull Public_IPv4_DNS/nginx:latest
```

Congratulations! You have successfully set up a private Docker registry on AWS EC2 instances, ensuring secure image storage and retrieval. Feel free to customize the configuration according to your specific requirements.
