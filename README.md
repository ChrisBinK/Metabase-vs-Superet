# Metabase-vs-Superet

There are many visualization tools on the market used for leveraging data insights. It is crucial to consider tools that can be efficiently utilized by data teams for data analysis or data stories. In this blog, we will focus on 2 of the most prominent tools; which are open-source and free, notably **Metabase** and **Apache Superset**.

Metabase and Superset are tools that allow for the creation of compelling visualizations for dashboards. Here are some key differences between the two: 
**Open Source**: 
- Metabase offers a free open-source tier and a paid version, while Superset is completely free open-source. 

**Visualization Gallery**: 
- Superset has a vast gallery of visualization tools, including charts, maps, network graphs, and more, which can be customized. 
- Metabase, however, has only a limited selection of visualization tools.

**Data Source**:
Both Superset and Metabase support a variety of databases and database connectors. Currently, Superset supports 42 of them, whereas Metabase supports 26. It's important to note that Superset works best with relational databases, while Metabase easily supports NoSQL databases such as MongoDB.

**Usability and Features**: 
- Metabase is simple and easy to use. It is designed for non-technical users and features a minimalistic and intuitive user interface. It is suitable for creating simple and quick dashboards. 
- On the other hand, Superset offers more comprehensive features such as security roles, authentication options, and customizable visualizations. It is designed for  the creation of complex data dashboards and is more suitable for technical users with a deep understanding of SQL.

**Usability and Features**: 
- Metabase is simple and easy to use. It is designed for non-technical users and features a minimalistic and intuitive user interface. It is suitable for creating simple and quick dashboards. 
- Superset offers a lot of features such as security roles, and authentication options. You can create customized visualizations. It is designed to create complex data dashboards and is more suitable for technical users with a deep understanding of SQL.

## Setting up Metabase on a RHEL based  VPS

In this section, we will walk through the steps required to install Metabase using Docker on a Virtual Private Server(VPS).
Step 1 **Connect to a VPS** using an SSH Client such as **PuTTy**

Step 2 **Install Docker** Refer to this guide [here](https://docs.docker.com/engine/install/) for other Linux Distributions.

```
# Update packages list
sudo dnf update -y

# Install docker and its dependencies
sudo dnf install install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

#Start docker service
sudo sudo systemctl start docker    
sudo systemctl enable docker 

```

Step 3 **Pull Metabase Docker Image**:
- Pull the metabase Docker image
```
    docker docker pull metabase/metabase
```

Step 4 ***Run Metabase Container***:
We need to create a container for Metabase, make sure to specify the desired port number where we will be able to access it
```
    docker run -d -p 3000:3000 --name metabase metabase/metabase
```

Step 5 **Check**  that Metabase Container is Working
Our container is available locally(on the server) at http://127.0.0.1:3000. To check if Metabase is accessible, run  `docker ps`, we will see a container called **metabase**. 
```
    curl -I http://127.0.0.1:3000
```

Step 6 Set up a **proxy reverse** in Apache Web Server
Open the configuration file in /etc/httpd/conf.d. We have to set up a reverse proxy to redirect our domain `www.example.com` to the Metabase container `http://127.0.0.1:3000`. Here is a sample code for the proxy reverse:  
```
    <VirtualHost *:443>
        ServerName www.example.com
        ProxyPreserveHost On
        ProxyPass / http://127.0.0.1:3000/
        proxyPassReverse  / http://127.0.0.1:3000/	
        Include /etc/letsencrypt/options-ssl-apache.conf
        SSLCertificateFile /etc/letsencrypt/live/www.example.com-005/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/www.example.com-005/privkey.pem
    </VirtualHost>
```

## Setting up Apache Superset on a RHEL based  VPS
In this section, we will walk through the steps required to install Superset using Docker on VPS.
Step 1 **Connect to a VPS** using an SSH Client such as **PuTTy**

Step 2 **Install Docker** . 
Make sure that docker is installed. Follw the previous section for the installation of Docker.

Step 3 **Create a Folder** 
Navigate to  `/var/www/html` and create a folder named superset-dashboard ` mkdir superset-dashboard`

Step 4 **Create a Docker compose file**
Create a  `docker-compose.yml` file and add the following yml code.
```
services:
  superset:
      build:
        context: ./superset
        dockerfile: dockerfile
      container_name: superset
      environment:
        - ADMIN_USERNAME=admin
        - ADMIN_EMAIL=myemail@domain.com
        - ADMIN_PASSWORD=admin
      ports:
        - '8088:8088'
```

Step 5 **Create a Docker file**

