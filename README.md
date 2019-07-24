# docker-sf-app
The goal of this app is run a symfony (3.4 & 4.x) app using docker & docker-compose in DEVELOP environment, this app is
not optimize to use in PRODUCTION environment.

NOTE: This app use [docker-services](https://github.com/ginsen/docker-services) to consume services as MySQL and others
network services.

## Install

Copy and rename .env.dist to .env and edit with your custom params.

```bash
$ cp .env.dist .env
```

Copy your SSH keys for access into the container.

```bash
$ cp ~/.ssh/id_rsa.pub ./docker/bash/ssh/
$ cp ~/.ssh/id_rsa ./docker/bash/ssh/
```

Download your symfony 4 application into this root, **with name app**

```bash
git clone https://github.com/symfony/your-symfony-app.git app
```

### SonarQube

If you has **sonar-project.properties.dist** in your project, clone it

```bash
$ cd app
$ cp sonar-project.properties.dist sonar-project.properties
```

And change Sonar server to `http://sonarqube:9000`,
such as ...

```bash
# ./app/sonar-project.properties

#----- SonarQube server
sonar.host.url=http://sonarqube:9000
```

### Xdebug
For use xdebug in your develop environment you need know what is your IP host on your IDE (PhpStorm) is running, for
this issue, run this command.

__If your has Ubuntu and awk installed, run it__

```bash
ip route|awk '/eno1 proto kernel/ { print $9 }'
```

You see your default network IP as __10.128.80.xxx__, copy your IP in your environment file (__.env__) as:

```bash
# .env

# Xdebug
# IP host when run your IDE (PhpStorm). Run in your shell: ip route|awk '/eno1 proto kernel/ { print $9 }'
IP_LISTEN_XDEBUG=10.128.80.5
XDEBUG_PORT=9001
```

Replace **10.128.80.5** with your default network IP and set **XDEBUG_PORT** with the port you choice.

### PHPStorm configuration Xdebug

Make sure you have the some port that you have configured previously in "XDEBUG_PORT" environment variable:

![alt text](https://github.com/ginsen/docker-services/raw/master/docker/doc/xdebug_port.png "Xdebug port")

Next, we need to configure a server. This is how PHPStorm will map the file paths in your local system to the ones in
your container.

Go to File -> Settings -> Languages and Frameworks -> PHP -> Servers

![alt text](https://github.com/ginsen/docker-services/raw/master/docker/doc/xdebug_server.png "Xdebug server")

Give a name to your server. It should match the value you have defined in your "APP" environment variable and replace 
"symfony-demo" with your "APP" name and replace **Port** 8888 with your "NGINX_PORT" environment variable.

## Up containers

Now you ar ready to up project with **docker-compose**.
Run this commands:

```bash
$ docker-compose up -d
```

Show containers is up.

```bash
$ docker-compose ps
```

Access into container

```bash
$ docker-compose exec web bash
```

When you access into container you access as root, you need change to normal user profile.

```bash
$ su {yourname}
```

Now you are into your project with your profile user, then you can run command to build your symfony app, remember edit 
the params of your .env app.

```bash
$ composer install
...
# yarn install
# ./bin/console doctrine:schema:create
# ... and other stuff
``` 

Or run test, sonar-scanner, ...

```bash
$ ./bin/phpunit
$ sonar-scanner
```