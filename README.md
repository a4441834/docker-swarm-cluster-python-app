Docker Swarm Cluster setup and testing the routing mesh:

Assumptions: 

a) Installed VirtuallBox and Docker desktop or docker toolkit, 
b) Have verified docker-hub login and ready, 
c) Able to pull and push the docker images against docker hub
d) Created an docker image with sample python REST apps by following steps given in this link ->https://docs.docker.com/get-started/part2/…
 

Step 1: Create VMs

$docker-machine create --driver virtualbox myvm1
$docker-machine create --driver virtualbox myvm2
$docker-machine create --driver virtualbox myvm3

Step 2: List VMs that are created successfully in the previous step 1
	
$docker-machine ls

Step 3:  Init swarm manager on myvm1 with port 2677 
                (Note: 2376 is deamon port not management port, where as 2377 is a swarm management port)

$docker-machine ssh myvm1 "docker swarm init --advertise-addr 192.168.99.103:2377"

Step 4: Join to the swarm leader , repeat the below steps for each node

$docker-machine ssh myvm1 "docker swarm join --token <token>  <myvm1-ip:2377"

$docker-machine ssh myvm2 "docker swarm join --token --token <token>  <myvm1-ip:2377"

$docker-machine ssh myvm3 "docker swarm join --token --token <token>  <myvm1-ip:2377"

Step 5: Verify the status of leader node and nodes

$docker-machine ssh myvm1 "docker node ls"

The status should be return as below 
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
xylk554239g5ycyzcgxx3jv8n *   myvm1               Ready               Active              Leader              19.03.1
w5eo29yhxiaf68xtgzi8zp3gc     myvm2               Ready               Active              Reachable           19.03.1
1h3oqhu6w0fk8fx239ucte70q     myvm3               Ready               Active              Reachable           19.03.1

Step 6: Set the Env. to leader node

$docker-machine env myvm1
$eval $(docker-machine env myvm1)
$docker-machine ls 

Status should be return as  :

$ docker-machine ls

NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER     ERRORS
myvm1   *        virtualbox   Running   tcp://192.168.99.103:2376           v19.03.1   
myvm2   -        virtualbox   Running   tcp://192.168.99.104:2376           v19.03.1   
myvm3   -        virtualbox   Running   tcp://192.168.99.105:2376           v19.03.1 

Step 7: deploy=>cd to the python apps root folder and run the command below to deploy stack against swarm cluster ( assuming that the python Dockerfil, app.py, docker-compose.yml and requirements.txt are exist in the folder while executing the below command )

 $docker stack deploy -c docker-compose.yml getstartedlab


Step 8: Verify the app stack deployment 

$docker stack ps getstartedlab

$ docker stack ps getstartedlab

ID                  NAME                   IMAGE                                 NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
9yimdjqhzyfv        getstartedlab_web.1    docker/image:tag   myvm2               Running             Running 54 seconds ago                       
0x1sedlkm800        getstartedlab_web.2    docker/image:tag   myvm2               Running             Running 54 seconds ago                       
yz37j5dv6wz8        getstartedlab_web.3    docker/image:tag   myvm3               Running             Running 56 seconds ago                       
7b7olq5ruzg6        getstartedlab_web.4    docker/image:tag   myvm1               Running             Running 52 seconds ago                       
ifbs5kve4hku        getstartedlab_web.5    docker/image:tag   myvm2               Running             Running 54 seconds ago                       
o3bf1bzxwu12        getstartedlab_web.6    docker/image:tag   myvm1               Running             Running 52 seconds ago                       
i5t6x55pu95q        getstartedlab_web.7    docker/image:tag   myvm3               Running             Running 56 seconds ago                       
1mqvgqkty8x6        getstartedlab_web.8    docker/image:tag   myvm2               Running             Running 54 seconds ago                       
xf2e7e2eprd1        getstartedlab_web.9    docker/image:tag   myvm3               Running             Running 56 seconds ago                       
r7twu18h7izo        getstartedlab_web.10   docker/image:tag   myvm1               Running             Running 52 seconds ago

Step 9: Test the app url using swarm cluster ip  address with the port 4000 and continue to refresh the page, on each click the hostname name gets changed as you notice.

http://192.168.99.103:4000 

	Hello World!
	Hostname: 66aa9321f4c3
	Visits: cannot connect to Redis, counter disabled
	
	
Note : The above deployment works per the diagram provided in the link ->  https://docs.docker.com/engine/swarm/images/ingress-routing-mesh.png

￼￼￼

Step 10: To scale up or down 

Change the replica set value in the docker-compose file  from 10 to 20 or desired value and deploy the stack again
    deploy:
      #replicas: 10 
	replicas: 20 

$docker stack deploy -c docker-compose.yml getstartedlab
	
	output: Updating service getstartedlab_web (id: lsftt6yee5kwq08ntyt2vvqr8)

$ docker stack ps getstartedlab
ID                  NAME                   IMAGE                                 NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
9yimdjqhzyfv        getstartedlab_web.1    docker/image:tag   myvm2               Running             Running 14 minutes ago                       
0x1sedlkm800        getstartedlab_web.2    docker/image:tag   myvm2               Running             Running 14 minutes ago                       
yz37j5dv6wz8        getstartedlab_web.3    docker/image:tag   myvm3               Running             Running 14 minutes ago                       
7b7olq5ruzg6        getstartedlab_web.4    docker/image:tag   myvm1               Running             Running 14 minutes ago                       
ifbs5kve4hku        getstartedlab_web.5    docker/image:tag   myvm2               Running             Running 14 minutes ago                       
o3bf1bzxwu12        getstartedlab_web.6    docker/image:tag   myvm1               Running             Running 14 minutes ago                       
i5t6x55pu95q        getstartedlab_web.7    docker/image:tag   myvm3               Running             Running 14 minutes ago                       
1mqvgqkty8x6        getstartedlab_web.8    docker/image:tag   myvm2               Running             Running 14 minutes ago                       
xf2e7e2eprd1        getstartedlab_web.9    docker/image:tag   myvm3               Running             Running 14 minutes ago                       
r7twu18h7izo        getstartedlab_web.10   docker/image:tag   myvm1               Running             Running 14 minutes ago                       
o30mzhjyizk2        getstartedlab_web.11   docker/image:tag   myvm3               Running             Running 51 seconds ago                       
3ul7g9wjco29        getstartedlab_web.12   docker/image:tag   myvm3               Running             Running 51 seconds ago                       
qvc9gndakq8p        getstartedlab_web.13   docker/image:tag   myvm1               Running             Running 52 seconds ago                       
qdlbpatuj0iy        getstartedlab_web.14   docker/image:tag   myvm1               Running             Running 52 seconds ago                       
9b0o4975io06        getstartedlab_web.15   docker/image:tag   myvm2               Running             Running 52 seconds ago                       
r6m2mqdnv5k4        getstartedlab_web.16   docker/image:tag   myvm1               Running             Running 52 seconds ago                       
g3ye9ur4czpg        getstartedlab_web.17   docker/image:tag   myvm3               Running             Running 52 seconds ago                       
t55bukl2eaop        getstartedlab_web.18   docker/image:tag   myvm3               Running             Running 51 seconds ago                       
x1t0ad3dxs8l        getstartedlab_web.19   docker/image:tag   myvm2               Running             Running 52 seconds ago                       
m1gu0pu4eue9        getstartedlab_web.20   docker/image:tag   myvm2               Running             Running 52 seconds ago 


To destroy the stack:

$docker stack rm getstartedlab                                                         # underlay stack from swarm cluster
$docker-machine ssh myvm2 "docker swarm leave —force”          # leave from swarm cluster
$docker-machine ssh myvm1 "docker swarm leave --force"            #leave from swarm cluster
$eval $(docker-machine env -u)                                                         # unset env
$docker-machine rm myvm1  -y                                                           # Remove docker machines/ vms
$docker-machine rm myvm2 -y
$docker-machine rm myvm3 -y
$docker-machine ls                                                                               # verify the cleanup

