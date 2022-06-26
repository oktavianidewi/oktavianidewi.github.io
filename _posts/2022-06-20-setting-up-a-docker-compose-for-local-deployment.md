---
layout: post
title:  "setting up a docker-compose for local deployment"
date:   2022-06-20 04:29:10 +0700
categories: docker
---

Hi there, 

In a working environment, we don’t always have a ‘safety place’ to test and play around with new tools, hence, doing experiment in local deployment still be our favorite things. Thankfully in this era, we have a tool named Docker, that make us easier to bring our production environment into the local computer. 

If you had experienced setting up a working environment locally in maybe 8 years ago, you would say thanks to the creator of Docker that save our life. 

I will give you an example, suppose you want to run a website up on [localhost](http://localhost), this website is built with PHP and has access to a local MySQL database. Thus, to build up your website, you need to activate the an Apache bundle server named XAMPP as PHP and a MySQL server. 

Your website is running on local server, then you make some changes on your local environment, test it and it works! But when you upload your site to the production server, your code is not working! Why? It is running on my local, bu not on production?

At this point, we need to do a lot of work ensuring that the whether PHP version, configuration and PHP extensions on XAMPP are the same as in the production? We will do a long journey and quite  frustrating process when tweaking the XAMPP to extend its capability to imitate the production server.

Please don’t do this. Just use docker instead. In docker, we can manage configuration of services by a single file.  

As a Data Engineer, I use multiple services at the same time, such as running a Golang application, postgreSQL, airflow, metabase, etc. Without docker, I would activate each service one by one.

In this article I want to explain how docker can help me to create a local environment and reduce unnecessary work in the future. I want to set up docker config and run a postgreSQL and metabase in one go with docker-compose. 

### docker-compose installation in Linux

Based on docker’s [documentation](https://docs.docker.com/get-started/08_using_compose/), docker-compose is a tool that was developed to help define and share multi-container applications. With Compose, we can create a YAML file to define the services and with a single command, can spin everything up or tear it all down.

Docker-compose is included in docker installation for Windows/Mac, but we have to install docker-compose separately in Linux

```bash
-- download docker-compose binary from github to /usr/local/bin
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

-- we made the docker-compose bin file executable 
sudo chmod +x /usr/local/bin/docker-compose

docker-compose --version
```

 

Here is a great article about difference between dockerfile and docker-compose [https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/Dockerfile-vs-docker-compose-Whats-the-difference](https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/Dockerfile-vs-docker-compose-Whats-the-difference)

### Create a compose file

In a docker-compose.yaml file, we might include settings 

- how much memory to assign
- security configuration
- how many virtual CPUs to allocate
- network settings
- host to Docker container port mappings
- where to store temporary files
- restart policies
- whether to limit read and write rates

### Config file for postgreSQL

```yaml
# the latest docker schema version
version: '3.7'

# list of services that we want to run
services:
  # name of the services
  postgres-db:
    image: postgres 
    restart: always
    ports: 
      - "5432:5432"
    environment:
      # https://hub.docker.com/_/postgres
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres_pass
      POSTGRES_DB: postgres
      PGDATA: /var/lib/postgresql/data
    volumes:
       - ./dir:/var/lib/postgresql/dir
       - ./data:/var/lib/postgresql/data
  
```

Here are the explanations of each config:

- `image: postgres` : we are going to download an official postgres image to local directory and run it in a docker container.
    - `image` is a read-only, immutable (unchangeable) file contains the source code, libraries, dependencies, tools, etc. for an application to run [https://phoenixnap.com/kb/docker-image-vs-container](https://phoenixnap.com/kb/docker-image-vs-container). Images are sometime referred to as snapshots.

![docker image and container](../../../../assets/img/docker-image-container.png)

[https://mariadb.com/wp-content/webp-express/webp-images/doc-root/wp-content/uploads/2022/02/es-docker-blog-img1.png.webp](https://mariadb.com/wp-content/webp-express/webp-images/doc-root/wp-content/uploads/2022/02/es-docker-blog-img1.png.webp)

- `restart: always` : it will `always` restart the container if it exits due to an error  [https://docs.docker.com/config/containers/start-containers-automatically/](https://docs.docker.com/config/containers/start-containers-automatically/)
- `ports: "5432:5432"` : declares that the local port `5432` is mapped to the internal port `5432` . `“localhost_port:internal_port”`
- `environment` : contains environment-variable setup to connect to postgres.
    - based on the postgresql image documentation ([https://hub.docker.com/_/postgres](https://hub.docker.com/_/postgres) ), if there is no database when `postgres` starts in a container, then `postgres` will create the default database for you.
- `volumes` are the preferred mechanism for persisting data generated by and used by Docker containers. `source_volume:target_volume` [https://docs.docker.com/storage/volumes/](https://docs.docker.com/storage/volumes/)

### Config file for metabase

Metabase is an open source business intelligence tool [https://www.metabase.com/docs/latest/users-guide/01-what-is-metabase.html](https://www.metabase.com/docs/latest/users-guide/01-what-is-metabase.html).  

```yaml
metabase-app:
    image: metabase/metabase
    restart: always  
    ports:
      - 3001:3000
    volumes:
      - ./metabase_data:/metabase-data
    environment:
      MB_DB_TYPE: postgres
      MB_DB_DBNAME: postgres
      MB_DB_PORT: 5432
      MB_DB_USER: postgres
      MB_DB_PASS: postgres_pass
      MB_DB_HOST: postgres-db
    depends_on:
      - postgres-db
    links: 
      - postgres-db
```

# Resources:

[https://www.metabase.com/docs/latest/operations-guide/running-metabase-on-docker.html](https://www.metabase.com/docs/latest/operations-guide/running-metabase-on-docker.html)

[https://www.alibabacloud.com/blog/practical-exercises-for-docker-compose-part-3_594415](https://www.alibabacloud.com/blog/practical-exercises-for-docker-compose-part-3_594415)

[https://martinahindura.medium.com/how-to-setup-metabase-with-postgresql-and-docker-compose-bf4cc7e7f899](https://martinahindura.medium.com/how-to-setup-metabase-with-postgresql-and-docker-compose-bf4cc7e7f899)