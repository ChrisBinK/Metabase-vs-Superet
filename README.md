![imagename](/images/apache%20vs%20metabase.jpeg)

# Metabase-vs- Apache Superset

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

Navigate to  `/var/www/html/superset-dashboard` and create a folder named superset ` mkdir superset`.
- In The folder, create a new **DockerFile**  with the follwing code:

```
    FROM apache/superset:4.0.0
    USER root
    RUN pip install mysqlclient

    ENV ADMIN_USERNAME $ADMIN_USERNAME
    ENV ADMIN_EMAIL $ADMIN_EMAIL
    ENV ADMIN_PASSWORD $ADMIN_PASSWORD

    RUN export FLASK_APP=superset

    COPY ./superset-init.sh /superset-init.sh
    COPY superset_config.py /app/
    ENV SUPERSET_CONFIG_PATH /app/superset_config.py

    RUN chmod 777 /superset-init.sh
    USER superset

    ENTRYPOINT [ "/superset-init.sh" ]
```

- Create another file `superset_config.py` with the following code
This is a file use to configure your application which will  override any of the parameters define here. It is better than  modify the core module.
Refer to [this link](https://superset.apache.org/docs/configuration/configuring-superset) for more information. 

```
    # Superset specific config
    ROW_LIMIT = 5000

   
    # Set `SECRET_KEY` environment variable.
    SECRET_KEY = 'YOUR_OWN_RANDOM_GENERATED_SECRET_KEY'

    # The SQLAlchemy connection string to your database backend. This connection defines the path to the database that stores your
    # superset metadata (slices, connections, tables, dashboards, ...).
    #SQLALCHEMY_DATABASE_URI = 'sqlite:////superset.db?check_same_thread=false'

    # Flask-WTF flag for CSRF
    WTF_CSRF_ENABLED = True

    # Add endpoints that need to be exempt from CSRF protection
    WTF_CSRF_EXEMPT_LIST = []

    # A CSRF token that expires in 1 year
    WTF_CSRF_TIME_LIMIT = 60 * 60 * 24 * 365

    # Set this API key to enable Mapbox visualizations
    MAPBOX_API_KEY = ''
    FEATURE_FLAGS = {
        "ENABLE_TEMPLATE_PROCESSING": True,
    }

    ENABLE_PROXY_FIX = True
```


- Lastly, create a file `superset-init.sh` with the followwing Bash script

This file has some command required to run for setting up Supperset application.

```
    #!/bin/bash

    # create Admin user, you can read these values from env or anywhere else possible
    superset fab create-admin --username "$ADMIN_USERNAME" --firstname Superset --lastname Admin --email "$ADMIN_EMAIL" --password "$ADMIN_PASSWORD"

    # Upgrading Superset metastore
    superset db upgrade

    # setup roles and permissions
    superset superset init 

    # Starting server
    /bin/sh -c /usr/bin/run-server.sh
```
Step 6  ***Create a docker image**

We need to create a docker container  with `docker compose build`  will build the services in the docker-compose.yml file.
Then `docker compose up -d`  to start the application and all the service.

Step 7 Set up a **proxy reverse** in Apache Web Server
Open the configuration file in /etc/httpd/conf.d. We have to set up a reverse proxy to redirect our domain `www.example.com` to the Metabase container `http://127.0.0.1:8088`. 

```
    <VirtualHost *:443>
        ServerName www.example.com
        ProxyPreserveHost On
        ProxyPass / http://127.0.0.1:8088/
        proxyPassReverse  / http://127.0.0.1:8088/	
        Include /etc/letsencrypt/options-ssl-apache.conf
        SSLCertificateFile /etc/letsencrypt/live/www.example.com-005/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/www.example.com-005/privkey.pem
    </VirtualHost>
```
We can be able to access Apache Superset on `www.example.com`


## Creating dashboard using Metabase.

- To create a dashboard, you need to set up an account on the server and login as show here.

![imagename](/images/Login%20metabase%20.png)

- To enable `.csv` file uplaod, you need to go to the admin setting and find the menu **Uploads**.

![imagename](/images/uplaod%20csv.png)

- In the main menu, click on![imagename](/images/add%20your%20database.png) to add a database of your choice. It is required even if you are using CSV.

- To upload a CSV, click on our collections

![imagename](/images/to_upload_csv.png)

- To create a **dashboard** in Metabase, Locate the menu Add new as shown here:

![imagename](/images/dashboard_menu_create.png)

- In Metabase, a dashboard is made of questions. A **question** is a query, its result, and visualizations such as Pie charts, bar charts, maps, etc. Let us create a question for our board.
Let us create a question by **clicking on ask a new one**.

- Select a table where our data will come from.

- Apply some Filter and Summarize, which are functions such as Average() and Sum () to create our query.

![imagename](/images/visualise%20metatdata.png)

- Click on the **Visualize** button to create the chart suitable for our query. Save and add the question to the dashboard.

![imagename](/images/chart_meta.png)

- Create as many questions as required for the dashboard. Once we are done, we can publish our dashboard by clicking on the **share** button.

![imagename](/images/sharing_metabase_dashboard.png)



## Creating dashboard in SUperbase.

We are going to create a dashboard base on data available in csv format. Here are the steps:

- To create a dashboard in Apache Superset, you need to login the credentials we created in the **docker compose file** previously.

![imagename](/images/superset_login.png)

- To upload a CSV, click settings menu, select **Database Connections**. Select upload a CSV file or Add a Database

![imagename](/images/superset_adding_db.png)


- To create a **dashboard** in Superset, Locate the menu **Charts** and click   **+add**. We can create as many chart needed for the dashboard.
![imagename](/images/superset_create_chart.png)

- Proceed to click on menu **Dashboard** and add all the cahrt we have created by dragging and dropping them on the grid.

![imagename](/images/superset_dashboard.png)





