# Docker-tut

Dependencies:
yum list installed | egrep "yum-utils|device-mapper-persistent-data|lvm2"

Create docker repo [can be installed by downloading RPM as well https://download.docker.com/linux/centos/7/x86_64/stable/Packages/]
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
grep -i enabled /etc/yum.repos.d/*

Install docker:
yum install docker-ce
yum list docker-ce --showduplicates | sort -r
systemctl start docker
systemctl enable docker

Create docker [non-sudo] users:
useradd pin
usermod -aG docker pin [docker group is created when docker is installonlyed]

install Docker-compose
curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --version
docker-compose -f docker-compose-prod.yml up --build


Running sample containers [su - pin]:
docker run hello-world
docker image ls
docker container ls --all


docker build -t friendlyhello .
docker login [provide docker hub credentials]
docker tag friendlyhello bharathhj/tutorial1:part2
docker push bharathhj/tutorial1:part2
docker run -d -p 8080:80 friendlyhello . [from path which has the dockerfile]

docker swarm init
docker stack deploy -c docker-compose.yml getstartedlab
docker service ps getstartedlab_web
docker service ls
docker container ls -q

docker stack rm getstartedlab_web
docker swarm leave --force


docker-compose.yml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:

Network types:
- Bridge - when your applications run in standalone containers that need to communicate [best when you need multiple containers to communicate on the same Docker host] - automatic service discovery [host name resolution wgen hosts are connected to used defined bridges...]
docker network create --driver bridge alp-net
docker network inspect alp-net
docker run -dit --name alp1 --network alp-net alpine ash
docker run -dit --name alp2 --network alp-net alpine ash
docker run -dit --name alp3 alpine ash (connects to default network bridge)
docker run -dit --name alp4 --network alp-net alpine ash
docker network connect bridge alp4 (only one network command can be provided with docker run, so need to add second network manually)

- host - used for swarm services, removes n/w isolation b/w containers [best when the network stack should not be isolated from the Docker host, but you want other aspects of the container to be isolated]
docker run -dit --network host --name my_nginx nginx

- overlay - connect multiple Docker daemons together and enable swarm services to communicate with each other [best when you need containers running on different Docker hosts to communicate, or when multiple applications work together using swarm services]
manager1:
docker swarm init --advertise-addr=10.212.152.23
docker network create -d overlay nginx-net
docker service create --name my-nginx --publish target=80,published=80 --replicas=5 --network nginx-net nginx

worker1:
docker swarm join --token SWMTKN-1-2fgcze9gh8u4hfyyedrdqm4723698eoasu2ofijlm74olb12q7-5m1223eem19rtpegaju95a2kw --advertise-addr 10.212.152.24 10.212.152.23:2377

	
- macvlan - Macvlan networks allow you to assign a MAC address to a container, making it appear as a physical device on your network. The Docker daemon routes traffic to containers by their MAC addresses [best when you are migrating from a VM setup or need your containers to look like physical hosts on your network, each with a unique MAC address]
- none - disable all networking for container
- plugins - allow you to integrate Docker with specialized network stacks





Storage types:
- Volumes
stored in a part of the host filesystem which is managed by Docker (/var/lib/docker/volumes/)
docker volume create my-vol
docker run -d --name devtest --mount source=myvol2,target=/app nginx:latest
docker service create -d --replicas=4 --name devtest-service --mount source=myvol2,target=/app,readonly nginx:latest
  
- Bind mounts
stored anywhere on the host system. They may even be important system files or directories. Non-Docker processes on the Docker host or a Docker container can modify them at any time
docker run -dit --name devtest --mount type=bind,source="$(pwd)"/target,target=/app,readonly nginx:latest
  
- tmpfs
stored in the host system’s memory, never written to the host system’s filesystem
docker run -dit --name tmptest --mount type=tmpfs,destination=/app nginx:latest



Date types:


https://docs.docker.com/develop/develop-images/multistage-build/#name-your-build-stages
https://docs.docker.com/engine/reference/commandline/attach/#description

Docker compose:
sudo curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version

Swarm and services:
docker swarm init --advertise-addr 10.212.152.23
docker swarm join-token worker
docker swarm join-token manager
docker service create --replicas 1 --name helloworld alpine ping docker.com
docker service scale <service_ID>=2

docker service create --replicas=3 --name=redis --update-delay=10s redis:3.0.6
docker service update --image redis:3.0.7 redis

docker node update --availability drain <node_name>
docker node promote node-3 node-2
docker node update --role manager/manager

key-value store holds information about the network state which includes discovery, networks, endpoints, IP addresses, and more. Docker supports Consul, Etcd, and ZooKeeper key-value stores. This example uses Consul.

https://docs.docker.com/machine/overview/#what-is-docker-machine
https://docs.docker.com/engine/swarm/ingress/#publish-a-port-for-a-service



/etc/haproxy/haproxy.cfg

global
        log /dev/log    local0
        log /dev/log    local1 notice
...snip...

# Configure HAProxy to listen on port 80
frontend http_front
   bind *:80
   stats uri /haproxy?stats
   default_backend http_back

# Configure HAProxy to route requests to swarm nodes on port 8080
backend http_back
   balance roundrobin
   server node1 192.168.99.100:8080 check
   server node2 192.168.99.101:8080 check
   server node3 192.168.99.102:8080 check
   
   
   ===========
   
   Commands
   
   [pin@labs-cent7-sbo ~]$ history | grep docker
    1  docker run -p 8080:8080 bharathhj/tutorial1:part3
    2  docker run -p 8080:80 bharathhj/tutorial1:part3
    3  docker images --all
    4  docker images ls --all
    5  docker image ls --all
    6  docker run friendlyhello
   13  cd /app/docker/
   15  vi docker-compose.yml
   16  docker run friendlyhello
   18  docker run friendlyhello
   19  docker run -p 8080:80 friendlyhello
   20  docker login
   21  docker run -p 8080:80 friendlyhello
   22  docker image ls
   23  docker run -p 8080:80 bharathhj/tutorial1:part3
   26  cd docker/
   35  docker logout
   37  cd /app/docker/
   41  cd /app/docker/29Mar2018/
   47  cd /app/docker/29Mar2018/
   48  docker build -t staticPage:v1 .
   49  docker build -t staticpage:v1 .
   50  cd /app/docker/29Mar2018/
   53  docker build -t staticpage:v1 .
   54  docker login
   57  docker hello-world
   58  docker run hello-world
   59  docker pull alpine
   60  docker imagee ls
   61  docker image ls
   62  docker image ls | grep alpi
   63  docker run alpine ls -l
   64  docker run alpine /bin/sh
   65  docker run -it alpine /bin/sh
   66  docker ps
   67  docker ps -a
   70  docker port static-site
   71  docker images
   72  docker run -d bharathhj/tutorial1
   73  docker run -d bharathhj/tutorial1:part2
   74  docker container
   75  docker container  ls
   76  docker container stop d874c2007de9
   77  docker container  ls
   78  docker run -d -p 8080:80 --name site_page bharathhj/tutorial1:part2
   79  docker port site_page
   80  docker run -d -p 8080:80 --name site_page bharathhj/tutorial1:part2
   81  docker run -d -p 80:80 --name site_page bharathhj/tutorial1:part2
   82  docker run -d -p 8080:80 --name site_page1 bharathhj/tutorial1:part2
   83  docker run -d -p 80:80 --name site_page1 bharathhj/tutorial1:part2
   84  docker container ls
   87  docker build -t staticpage:v1 .
   89  docker build -t staticpage:v1 .
   93  cd /app/docker/29Mar2018/
   95  docker build -t staticpage:v1 .
   97  docker build -t staticpage:v1 .
   98  docker push staticpage:v1
   99  docker login
  100  docker push staticpage:v1
  102  docker push bharathhj/staticpage:v1
  103  docker build -t bharathhj/staticpage:v1 .
  104  docker push bharathhj/staticpage:v1
  105  docker run -it bharathhj/staticpage:v1 /bin/bash
  108  docker run -i -d alpine sh
  109  docker run -it -d alpine sh
  110  docker run -it alpine sh
  111  docker container
  112  docker container ls
  113  docker container stop d5dc9c373cda 39f44f36a1cf
  114  docker container ls
  115  docker image ls
  116  docker run -it bharathhj/staticpage:v1
  118  docker run -it bharathhj/staticpage:v1
  123  docker run -it bharathhj/staticpage:v1
  127  docker run -it bharathhj/staticpage:v2
  128  docker build -t bharathhj/staticpage:v2 .
  130  docker info
  131  docker version
  132  docker search
  133  docker search ubuntu
  136  docker container ls
  138  cd /app/docker/29Mar2018/
  140  docker login
  141  docker run alpine "ls -l /"
  142  docker run alpine "ls -l"
  143  docker run alpine
  144  docker -it run alpine
  145  docker run -it alpine
  146  docker search apache
  147  docker inspect alpine
  148  docker run -it alpine
  149  docker image history
  150  docker image history alpine
  151  docker image history staticpage:v1
  152  docker image history alpine
  154  docker history alpine
  155  docker container ls
  156  docker container kill 84189902c3de
  157  docker run -d alpine
  158  docker container ls
  159  docker run -d bharathhj/tutorial1:part2
  160  docker container ls
  161  docker container kill 3a001a8f0ab8
  162  docker container ls
  163  docker run -d -p 8080:80 bharathhj/tutorial1:part2
  164  docker network ls
  165  docker container ls
  166  docker attach 843a0e4c0396
  167  docker search APACHE
  168  docker run -d httpd
  169  docker image ls
  170  docker images ls
  171  docker image ls
  172  docker search http
  173  docker pull httpd
  174  docker image ls
  175  docker run -d -it httpd
  176  docker container ls
  177  docker network ls
  178  docker attach 3994f00d6e6c
  179  docker image ls
  180  docker container ls
  181  docker container stop 7ba47237dc8e
  182  docker container ls
  183  docker image ls
  184  docker container ls
  185  docker run -it httpd
  187  cd /app/docker/
  189  docker volume create
  191  docker volume ls
  194  docker login
  198  docker build -f Dockerfile
  199  docker build -f Dockerfile .
  202  docker image ls
  203  docker image rm 028ebd4261a4 028ebd4261a4 8b625a9bc0fe 9b5159f647d8 e8a8eec06244 3c3067bb7cc7 77f9d2fa8385 805130e51ae9 1816fe2c7b42
  204  docker image rmi 028ebd4261a4 028ebd4261a4 8b625a9bc0fe 9b5159f647d8 e8a8eec06244 3c3067bb7cc7 77f9d2fa8385 805130e51ae9 1816fe2c7b42
  205  docker image --help
  206  docker image rm --help
  207  docker image rmi -f 028ebd4261a4 028ebd4261a4 8b625a9bc0fe 9b5159f647d8 e8a8eec06244 3c3067bb7cc7 77f9d2fa8385 805130e51ae9 1816fe2c7b42
  208  docker image ls
  209  docker container ls
  210  docker run -d -p 8080:80 friendlyhello
  211  docker container ls
  212  docker network ls
  213  docker network inspect bridge
  216  cd /app/docker/29Mar2018/
  217  docker network ;s
  218  docker network ls
  219  docker container ls
  221  docker container ls
  239  docker container ls
  241  docker load --help
  242  docker network
  243  docker network ls
  253  docker container ls
  254  docker attach f79c2c836fb2 -it
  255  docker attach f79c2c836fb2
  256  docker image ls
  258  docker run -it alpine /bin/bash
  259  docker run -it alpine
  260  docker volume create my-vol1
  261  docker volume ls
  262  docker volume inspect my-vol1
  263  docker run -d
  264  docker run -d --name volTest
  265  docker run -d --name volTest --mount source=my-vol1,target=/app appine
  266  docker run -d --name volTest --mount source=my-vol1,target=/app alpine
  267  docker inspect volTest
  268  docker container ls
  269  docker containers ls
  270  docker container ls --all
  271  docker containers ls
  272  docker container ls
  273  docker container stop voltest
  274  docker container rm voltest
  275  docker volume ls
  276  docker volume rm my-vol1
  277  docker volume rm -f my-vol1
  278  docker inspect my-vol1
  279  run docker -it --name voltest2 --mount source=myVol2,target=/app alpine
  280  docker run -it --name voltest2 --mount source=myVol2,target=/app alpine
  281  docker inspect voltest2
  282  docker inspect myVol2
  283  docker inspect voltest2
  284  docker inspect voltest2 --help
  285  cd /var/lib/docker/volumes/myVol2/_data
  293  docker run -it --name voltest3 --mount type=bind,source=/app/symc/,target=/app/symc alpine
  294  docker container rm vol*
  295  docker container rm volTest voltest2 voltest3
  298  docker run -it --name voltest3 --mount type=bind,source=/app/symc/,target=/app/symc alpine
  299  cd /app/docker/
  305  docker run -it --name voltest3 --mount type=bind,source=/app/symc/,target=/app/symc alpine
  306  docker container stop voltest3
  307  docker container rm voltest3
  308  docker run -it --name voltest3 --mount type=bind,source=/app/symc/,target=/app/symc alpine
  310  docker volume rm -f my-vol1
  311  cd /app/docker/
  317  docker ps -s
  318  docker run -d alpine
  319  docker ps -s
  320  docker container ls
  338  docker build -t bharath_hj/hellohost:1.0 -f Dockerfile
  339  docker build -t bharath_hj/hellohost:1.0 -f Dockerfile .
  340  docker image ls
  341  docker history 466109e04e56
  342  docker image ls
  343  docker run bharath_hj/hellohost
  344  docker run bharath_hj/hellohost:1.0
  346  docker build -t bharath_hj/hellohost:1.1 -f Dockerfile .
  347  docker image ls
  348  docker history cf584afdd2eb
  349  docker run bharath_hj/hellohost:1.1
  351  docker inspect cf584afdd2eb
  353  docker build -t bharath_hj/hellohost:1.2 -f Dockerfile .
  354  docker run bharath_hj/hellohost:1.2
  358  vi /app/docker/20Apr2018/hello.sh
  359  /app/docker/20Apr2018/hello.sh
  361  cd docker/
  366  docker info
  367  docker info | grep -i storage
  368  dockerd
  370  cd /app/docker/
  375  docker build -t help1 -f Dockerfile .
  376  docker image ls
  377  docker run -it help1 /bin/bash
  378  docker run help1
  379  cd /app/docker/
  381  vi docker-compose.yml
  383  cd /app/docker/
  392  vi docker-compose.yml
  393  cat docker-compose.yml
  394  vi docker-compose.yml
  396  cd /app/docker/29Apr2018/
  397  docker build
  398  docker build --help
  399  docker build --file  docker-compose.yml .
  400  vi docker-compose.yml
  403  cd /app/docker/
  405  cat docker-compose.yml
  406  cd /app/docker/29Apr2018/
  408  cp docker-compose.yml docker-compose-prod.yml
  410  vi docker-compose-prod.yml
  411  docker-compose docker-compose-prod.yml up --build
  412  docker-compose -f docker-compose-prod.yml up --build
  413  vi docker-compose-prod.yml
  414  docker-compose -f docker-compose-prod.yml up --build
  415  vi docker-compose-prod.yml
  416  docker-compose -f docker-compose-prod.yml up --build
  417  cd /app/docker/29Apr2018/
  418  docker-compose up --build
  419  vi docker-compose.yml
  420  docker-compose up --build
  421  cat docker-compose-prod.yml
  422  vi docker-compose-prod.yml
  423  cd /app/docker/
  431  docker build -t bharath_hj/scratchv1.0 -f Dockerfile .
  432  docker images ls
  433  docker image ls
  434  docker image ls | grep scratch
  435  docker run -it bharath_hj/scratchv1.0 /bin/bash
  436  docker run -it bharath_hj/scratchv1.0
  437  docker run --help
  438  docker run bharath_hj/scratchv1.0
  439  cd /app/docker/29Apr2018/
  441  docker-compose up --build
  443  docker run -dit --network host --name my-nginx nginx
  447  docker container ls
  448  docker stop nginx
  449  docker container stop nginx
  450  docker container stop my-nginx
  451  docker container rm my-nginx
  452  docker container ls
  453  docker run -dit --network host --name my-nginx nginx
  454  docker --version
  456  docker container ls
  457  docker run -dit --name alp1 alpine ash
  458  docker run -dit --name alp2 alpine ash
  459  docker container ls
  460  docker network inspect bridge
  461  docker attach alp1
  465  docker attach alp1
  466  docker container ls
  467  docker run -dit --name alp1 alpine ash
  468  docker container ls -a
  469  docker container rm alp1
  470  docker container rm alp2
  471  docker container stopap2
  472  docker container stop alp2
  473  docker container rm alp2
  474  docker container ls
  475  docker run -dit --name alp1 alpine ash
  476  docker run -dit --name alp2 alpine ash
  477  docker network inspect bridge
  478  docker attach alp1
  479  docker container ls
  480  docker container stop alpine1 alpine2
  481  docker container stop alp1 alp2
  482  docker container rm alp1 alp2
  483  docker network create --driver bridge alp-net
  484  docker network ls
  485  docker network inspect alp-net
  486  docker network inspect
  487  docker network inspect bridge
  488  docker network inspect alp-net
  489  docker run -dit --name alp1 --network alp-net alpine ash
  490  docker run -dit --name alp2 --network alp-net alpine ash
  491  docker run -dit --name alp3 alpine ash
  492  docker run -dit --name alp4 --network alp-net alpine ash
  493  docker network connect bridge alp4
  494  docker container ls
  495  docker network inspect bridge
  496  docker network inspect alp-net
  497  docker network inspect alp-net |e grep "Name|IPv4"
  498  docker network inspect alp-net | egrep "Name|IPv4"
  499  docker network inspect bridge | egrep "Name|IPv4"
  500  docker attach alp1
  501  docker attach alp4
  502  docker container stop alp1 alp2 alp3 alp4
  503  docker container rm alp1 alp2 alp3 alp4
  504  docker network rm alp-net
  505  docker info
  506  docker search --help
  507  docker search tomcat
  508  docker --version
  509  docker container ls
  510  docker swarm init
  511  docker swarm leave
  512  docker swarm init
  513  docker swarm leav
  514  docker swarm leave
  515  docker swarm leave --force
  516  docker swarm init --advertise-addr=10.212.152.23
  517  docker node ls
  518  docker node ls --filter status=ready
  519  docker node ls --filter role=manager
  520  docker node ls --filter role=worker
  521  docker network ls
  522  docker network create -d overlay nginx-net
  523  docker service create \
  524  docker container ls
  525  docker service create --name my-nginx --publish target=80,published=80 --replicas=5 --network nginx-net nginx
  526   docker service ps ojqugkt9wb8edzqduyvyu51nn
  527  docker service rm ojqugkt9wb8edzqduyvyu51nn
  528   docker service ps ojqugkt9wb8edzqduyvyu51nn
  529  #docker service create --name my-nginx --publish target=80,published=80 --replicas=5 --network nginx-net nginx
  530  docker container ls
  531  docker container stop 53170a93b314
  532  docker container rm 53170a93b314
  533  docker container ls
  534  docker service create --name my-nginx --publish target=80,published=80 --replicas=5 --network nginx-net nginx
  535   docker service ls
  536  docker network inspect nginx-net
  537  docker service inspect my-nginx
  538  docker network create -d overlay nginx-net-2
  539  docker service update --network-add nginx-net-2 --network-rm nginx-net my-nginx
  540   docker service ls my-nginx
  543  docker node ls
  544  docker network ls
  546  docker container ls
  547  docker run -dit --name new1 alpine ash
  548  docker container ls
  549  docker network inspect bridge
  550  docker network inspect nginx-net
  551  docker container inspect new1
  552  docker ps
  553  docker
  555  docker container ls
  556  docker run -dit --name new1 alpine ash
  557  docker container ls
  558  docker network inspect bridge
  559  docker network inspect nginx-net
  560  docker container inspect new1
  561  docker ps
  562  docker
  564  history | grep -i docker-compose
  565  ll /var/lib/docker/volumes/
  569  docker volume create myVolume1
  570  docker run -dit --name alpVolume --mount source=myVolume1,target=/app alpine ash
  571  docker attach alpVolume
  573  cd /app/docker/
  580  docker --version
  581  cd /app/docker/07May2018/
  586  docker build -t hello_1 -f Dockerfile .
  591  docker build -t hello_1 -f Dockerfile .
  593  docker build -t hello_1 -f Dockerfile .
  594  docker run hello_1
  595  docker image ls
  597  docker build -t hello_1 -f Dockerfile .
  598  docker run -dit hello_1 ash
  599  docker container ls
  600  docker attach 573c18c6beb3
  601  docker container ls
  607  docker build -t hello_1 -f Dockerfile .
  608  docker image ls
  609  docker run -dit hello_1 ash
  610  docker container ls
  611  docker container stop 573c18c6beb3
  612  docker container rm 573c18c6beb3
  613  docker attach bc817cf3a7cb
  614  docker container ls
  615  docker run hello_1 ash
  617  ll /var/lib/docker/
  622  docker image ls
  623  dockerd
  625  docker container ls
  626  docker stats 01d688725454
  628  cd /app/docker/
  636  docker search --help
  637  docker search --filter="tomcat"
  638  docker search --filter "tomcat"
  640  docker search tomcat
  641  docker swarm leave
  642  docker info | grep swarm
  643  docker swarm init --advertise-addr="10.212.152.23"
  644  docker container ls
  645  docker node ls
  646  docker container ls
  647  docker swarm leave
  648  docker swarm leave --force
  649  docker node ls
  650  docker container ls
  651  docker container rm 01d688725454
  652  docker container ls
  653  docker container stop 01d688725454
  654  docker container ls
  655  docker container rm 01d688725454
  656  docker container ls
  657  docker swarm init --advertise-addr 10.212.152.23
  658  docker info | grep -i swarm
  659  docker node ls
  660  docker swarm join-token worker
  661  docker service ls
  662  docker service create --replicas 1 --name helloworld alpine ping docker.com
  663  docker service ls
  664  docker service inspect wy5kxzwjwcec
  665  docker service inspect --pretty wy5kxzwjwcec
  666  docker service ps wy5kxzwjwcec
  667  docker service inspect --pretty wy5kxzwjwcec
  668  docker service ps wy5kxzwjwcec
  669  docker service inspect --pretty wy5kxzwjwcec
  670  docker service ps wy5kxzwjwcec
  672  cd /app/docker/
  674  docker node ls
  675  docker service ls
  676  docker service scale wy5kxzwjwcec=2
  677  docker service ls
  678  docker node ls
  679  docker service ps helloworld
  681  docker service ps helloworld
  682  docker service ps helloworld --filter=running
  683  docker service ps helloworld --filter state=running
  684  docker node ls
  685  docker ps
  686  docker service rm helloworld
  687  docker ps
  688  docker service rm helloworld
  689  docker node ls
  690  docker service ls
  691  docker service create --replicas=3 --name=redis --update-delay=10s redis:3.0.6
  692  docker node ls
  693  docker service ls redis
  694  docker service ps redis
  695  docker service inspect redis
  696  docker service inspect redis | grep -i redis
  697  docker network inspect
  698  docker network inspect bridge
  699  docker network ls
  700  docker network inspect ingress
  701  docker network inspect overlay
  702  docker network inspect host
  703  docker service update --image redis:3.0.7 redis
  704  docker service ps redis
  705  docker service scale redis=4
  706  docker service ps redis
  707  docker node ls
  708  docker node inspect --pretty
  709  docker node inspect --pretty g611ke0q66qs0yjj0mcf2lsgb
  710  docker node inspect --pretty dp21g0cxmhbrxkbne0wsydixm
  711  docker node ls
  712  docker node update --availability drain labs2-cent7-sbo
  713  docker node ls
  714  docker node inspect --pretty labs2-cent7-sbo
  715  docker service ps redis
  716  docker node update --availability active labs2-cent7-sbo
  717  docker node inspect --pretty labs2-cent7-sbo
  718  docker service ps redis
  719  docker node inspect --pretty labs2-cent7-sbo
  720  docker node ls
  721  docker node update --availability drain labs2-cent7-sbo
  722  docker node ls
  723  docker node update --availability active labs2-cent7-sbo
  724  docker node ls
  725  docker node update --availability pause labs2-cent7-sbo
  726  docker node ls
  727  docker node update --availability active labs2-cent7-sbo
  728  docker node ls
  729  docker node rm labs2-cent7-sbo
  730  docker node update --availability drain labs2-cent7-sbo
  731  docker node rm labs2-cent7-sbo
  732  docker service rm redis
  733  docker service ls
  734  docker service ps
  735  docker service ps redis
  736  docker service ps --help
  737  docker service ps
  738  docker node ls
  739  docker node rm labs2-cent7-sbo
  740  docker node ls
  741  docker swarm leave
  742  docker node ls
  743  docker node rm labs2-cent7-sbo
  744  docker node ls
  745  docker swarm leave --force
  746  docker node ls
  747  docker image ls
  748  docker swarm init --advertise-addr 10.212.152.23
  749  docker service create --replicas=2 --name=helloHost bharath_hj/hellohost:1.2
  751  cd /app/docker/
  755  docker container ls
  756  docker node ls
  757  docker node update --state=drain
  759  docker node ls
  760  docker swarm leave
  761  docker swarm leave --force
  762  docker node ls
  764  cd /app/docker/
  772  vi docker-compose.yml
  773  diff docker-compose-prod.yml docker-compose.yml
  774  cat docker-compose.yml
  775  history | grep ^docker-compose
  777  cd /app/docker/
  782  vi docker-compose.yml
  784  vi docker-compose.yml
  786  docker service ps
  787  docker service ls
  788  docker service create --replicas=2 --name=helloHost bharath_hj/hellohost:1.2
  789  docker swarm init
  790  docker node ls
  791  docker node inspect labs-cent7-sbo
  795  docker swarm init --help
  796  docker node ls
  797  docker node inspect sbl03v83okkh4701csawsb7fh | grep -i token
  798  docker swarm init --help
  799  history | grep -i ^docker | grep listen
  801  cd /app/docker/
  803  docker node ls
  804  docker node inspect labs-cent7-sbo
  805  docker swarm join-token worker
  806  docker node ls
  807  docker node inspect labs2-cent7-sbo | grep -i token
  808  docker node inspect labs2-cent7-sbo
  809  vi docker_worker.crt
  810  diff dock.crt docker_worker.crt
  811  docker service ps
  812  docker service ls
  813  docker swarm join-token worker
  814  docker swarm join-token manager
  815  docker node ls
  816  docker node rm 5iim83se952bftw7ak76jzb2c
  817  docker node ls
  818  docker node inspect self --pretty
  819  cd /app/docker/
  821  docker node ls
  822  docker node inspect sbl03v83okkh4701csawsb7fh
  823  docker node inspect --pretty sbl03v83okkh4701csawsb7fh
  824  docker node inspect sbl03v83okkh4701csawsb7fh --format "{{ .ManagerStatus.Reachability }}"
  826  cd /app/docker/
  828  docker node ls
  829  docker node inspect --pretty labs-cent7-sbo
  831  cd /app/docker/
  835  cd /app/docker/
  841  docker container ls
  842  docker run -rm -dit --name basic1 alpine ash
  843  docker history
  844  docker history --help
  847  docker run rm -dit --name basic1 alpine ash
  848  docker run --help
  849  docker run --rm -dit --name basic1 alpine ash
  850  docker container ls
  851  docker attach f43b58ec3a9b
  852  docker stop basic1
  853  docker container ls
  854  docker image ls
  855  docker run --rm -dit --name basic1 --mount type=bind,source="$(pwd)",target=/app alpine ash
  856  docker container ls
  857  docker container ls --help
  858  docker container ls -q
  859  docker container ls -s
  860  docker container ls -qs
  861  docker container ls -f ID
  862  docker container ls --help
  863  docker container ls -f=ID
  864  docker container ls
  865  docker attach  4a339c30fdcd
  866  docker container ls
  867  docker run --rm -dit --name basic1 --mount type=bind,source="$(pwd)",target=/app ;
  868  docker volume ls
  869  docker volume inspect myVolume1
  870  docker run --rm -dit --name basic1 --mount type=bind,source="$(pwd)",target=/app \
  871  docker run --rm -dit --name basic1 --mount type=bind,source="$(pwd)",target=/app alpine ash
  872  docker container ls
  873  docker attach 9f59abb1b064
  876  docker run --rm -dit --name basic1 --mount type=bind,source="$(pwd)",target=/app alpine ash
  877  docker container ls
  878  docker run --help
  879  docker run --help | grep user
  880  docker container ls
  881  docker container stop basic1
  882  docker container ls
  883  docker run --rm -dit --name basic1 --mount type=bind,source="$(pwd)",target=/app --user pin alpine ash
  884  docker container ls
  886  docker container ls
  888  docker container ls
  889  docker run --rm -dit --name basic1 --mount type=bind,source="$(pwd)",target=/app --user pin alpine ash
  890  docker run --rm -dit --name basic1 --mount type=bind,source="$(pwd)",target=/app --user:pin alpine ash
  891  docker run --rm -dit --name basic1 --mount type=bind,source="$(pwd)",target=/app --user pin:pin alpine ash
  892  docker run --rm -dit --name basic1 --mount type=bind,source="$(pwd)",target=/app --mount type=bind,source=/etc/passwd,target=/etc/passwd alpine ash
  893  docker attach basic1
  894  docker run --rm -dit --name basic1 --mount type=bind,source="$(pwd)",target=/app --mount type=bind,source=/etc/passwd,target=/etc/passwd --user pin:pin alpine ash
  895  docker stop basic1
  896  docker container ls
  898  docker run --rm -dit --name basic1 --mount type=bind,source="$(pwd)",target=/app --mount type=bind,source=/etc/passwd,target=/etc/passwd --user="pin" alpine ash
  899  docker
  901  cd /app/docker/
  902  docker node ls
  903  docker node rm s4blw5ngeb6esl7e9t8a7mtlb
  904  docker node rm s4blw5ngeb6esl7e9t8a7mtlb --force
  905  docker node ls
  907  cd /app/docker/
  909  docker run --help
  911  docker run hello-world
  912  cd /app/docker/
  923  docker build --help
  924  docker build --tag hello_sample1.0 --file Dockerfile .
  925  docker images ls | grep hello_sample
  926  docker images ls
  927  docker logout
  928  docker login
  929  docker pull hello_sample1.0:latest
  930  docker pull hello_sample1.0
  931  docker login
  932  docker pull hello_sample1.0
  933  docker pull hello_sample1.0:latest
  934  docker build --tag hello_sample1.0 --file Dockerfile .
  935  docker images ls
  936  docker images
  937  docker images --help
  938  docker images
  939  docker run hello_sample1.0
  940  docker run --help
  941  docker run --rm hello_sample1.0 -dit ash
  942  docker run --rm hello_sample1.0 -dit /bin/bash
  943  #docker run --rm hello_sample1.0 -dit /bin/bash
  945  #docker run --rm -dit --name hello_sample hello_sample1.0 ash
  946  docker run --rm -dit --name hello_sample hello_sample1.0 ash
  947  docker container ls
  948  docker attach ca6646c60b0b
  949  docker run --rm -dit --name hello_sample hello_sample1.0 ash
  950  docker attach 63f50692707dea946f5f15ef383d3a773794cf42d00ef48e96d6db5efee495fa
  951  docker run --rm -dit --name basic1 --mount type=bind,source="$(pwd)",target=/app --mount type=bind,source=/etc/passwd,target=/etc/passwd --user="pin" alpine ash
  960  docker container ls
  961  docker image ls
  962  docker run --rm --name sample alpine -dit ash
  963  docker run --rm -dit --name sample alpine ash
  964  docker container ls
  965  docker attach sample
  966  docker container ls
  967  docker attach sample
  968  docker container ls
  969  docker attach hello_sample
  970  docker container l
  971  docker container ls
  972  docker run --rm -dit --name sample alpine ash
  973  docker container stats
  974  docker container sample stats
  975  docker container ls
  976  docker container sample logs
  977  docker container logs
  978  docker container logs sample
  979  docker container logs --help
  980  docker container logs --details sample
  981  docker container logs -f sample
  982  docker container logs -t sample
  983  docker container logs -f sample
  987  docker node ls
  988  docker node leave
  989  docker swarm leave
  990  docker swarm leave --force
  991  docker node ls
  992  docker swarm init
  993  docker node ls
  994  docker swarm join-token worker
  995  docker swarm join-token manager
  996  docker node ls
  997  docker swarm join-token worker
  998  docker node ls
 1000  history | grep docker
[pin@labs-cent7-sbo ~]$

