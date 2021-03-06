# docker_start_app
I Persiting by using volume 
1 docker volume create todo-db
2 docker run -dp 3000:3000 -v todo-db:/etc/todos docker_start_app

what's going on inside?
docker volume inspect todo-db
[
    {
        "CreatedAt": "2019-09-26T02:18:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local"
    }
]

II network between containers
1 docker network create todo-app
windows use nat
docker network create --driver nat todo-app

2 docker run -d \
    --network todo-app --network-alias mysql \
    -v todo-mysql-data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=secret \
    -e MYSQL_DATABASE=todos \
    mysql:5.7
You'll notice we're using a volume named todo-mysql-data here and mounting it at /var/lib/mysql, which is where MySQL stores its data. However, we never ran a docker volume create command. Docker recognizes we want to use a named volume and creates one automatically for us.

III connect to mysql container
If we run another container on the same network, how do we find the container (remember each container has its own IP address)?
To figure it out, we're going to make use of the **nicolaka/netshoot** container, which ships with a lot of tools that are useful for troubleshooting or debugging networking issues.
1 docker run -it **--network todo-app** nicolaka/netshoot
2 inside the continer: dig mysql //mysql is the running mysql conatiner's network-alias
; <<>> DiG 9.14.8 <<>> mysql
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34446
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;mysql.                         IN      A

;; ANSWER SECTION:
mysql.                  600     IN      A       172.27.215.98

;; Query time: 4 msec
;; SERVER: 172.27.208.1#53(172.27.208.1)
;; WHEN: Sat Mar 21 05:11:44 UTC 2020
;; MSG SIZE  rcvd: 44

**not good idea to use ENV in production, there would be a risk of security problems.**
docker run -dp 3000:3000 \
  -w /app -v $PWD:/app \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"

docker run -dp 3000:3000 -w /app -v $PWD:/app --network todo-app -e MYSQL_HOST=mysql -e MYSQL_USER=root -e MYSQL_PASSWORD=secret -e MYSQL_DB=todos node:12-alpine  sh -c "yarn install && yarn run dev" 
