# kong-konga-docker-compose

Use the latest version of kong and konga, and do not need to compile, directly pull the official image to run.

Four containers:

* *kong-database* : postgres
* *kong-migrations* : kong
* *kong* : kong
* *konga* : konga



## Usage

```bash
docker-compose up -d
```



## Configure https

Configure https as follows, and then use https protocol to access port 8443.

```yaml
kong:
  ...
  volumes:
    - "./ssl:/mnt/ssl"
  environment:
    - KONG_SSL_CERT=/mnt/ssl/ssl.pem
    - KONG_SSL_CERT_KEY=/mnt/ssl/ssl.key
    ...
```



## If you are using docker-compose-production.yml

The production environment needs to ensure security and performance. Here, the production environment configuration file created in a simple scenario is for reference only. If you have a better way, please let me know.

#### 1. use tls certificate

Create a folder ssl under the same level folder of docker-compose-production.yml, built-in tls certificate file, the specific name can be modified in the yaml file, the default is `ssl.key` and `ssl.pem`.

#### 2. create network

Create a network for kong, and then connect the business nodes through the intranet. The default network name here is kong-net, you can create it with the following command.

```bash
docker network create kong-net
```

#### 3. run

```bash
docker-compose up -d
```

#### 4. register service and route of konga

In the production version, the port of konga is not exposed, so that konga's request also distributes traffic on kong. So here you need to manually request kong's management api for registration.

Install `curl` on any node in the network where kong is located, and then execute the following command.When both requests return `HTTP/1.1 201 Created`, the registration is successful.

```bash
# register service
curl -i -X POST --url http://kong:8001/services/ --data 'name=konga' --data 'url=http://konga:1337'
# register route
curl -i -X POST --url http://kong:8001/services/konga/routes --data 'name=konga' --data 'paths[]=/konga'
```

Then, go to the browser to visit your web address. The first time you log in to konga, you need to visit the following address.The key is to add the path of `/konga`, make it `/konga/register` path, the first time the `/register` path will be loaded directly by default.

```
https://<your-domain-name>/konga/register#!/services
```



## Official documentation

install kong using docker: https://docs.konghq.com/install/docker/
