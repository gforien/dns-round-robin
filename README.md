# dns-round-robin 💫♻


## Avec des containers existants
```sh
    # 2
    docker run --rm --name proxy --network net1 -d nginx:alpine
    docker network connect --alias round-robin net1 host1
    docker network connect --alias round-robin net1 host2
```

## Avec des nouveaux containers
```sh
    # 1
    docker network create net1 # [--driver bridge]

    # 2
    docker run --rm --name host1  --network net1 --network-alias host -d nginx
    docker run --rm --name host2  --network net1 --network-alias host -d nginx
    docker run --rm --name proxy --network net1 -d nginx:alpine
    docker network inspect net1

    # (3) nginx configuration
    docker exec -it host1 sh
    [host1] / # echo SRV1 > /usr/share/nginx/html/index.html
    [host1] / # exit
    docker exec -it host2 sh
    [host2] / # echo SRV2 > /usr/share/nginx/html/index.html
    [host2] / # exit

    # (4) proxy DNS lookup
    docker exec -it proxy sh
    [proxy] # / nslookup host1
    # 172.18.0.3
    [proxy] # / nslookup host2
    # 172.18.0.4
    [proxy] # / nslookup host
    # 172.18.0.3
    # 172.18.0.4

    # (5) load balancing
    [proxy] # / apk add vim
    [proxy] # / vim script.sh; chmod a+x script.sh
        #!/bin/sh
        total=5000
        c=0
        for i in `seq 1 $total`
                do if [ `curl -s host:80` = 'SRV1' ]
                then c=$((c+1))
                fi
        done
        echo "SRV1 = $c/$total"
    [proxy] # / ./script.sh
    # SRV1 = 2484/5000

    docker stats
    # CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
    # 280d27ef184d        host2               3.92%               2.719MiB / 3.848GiB   0.07%               2.06MB / 2.21MB     0B / 0B             3
    # 56439f22ae6f        host1               3.88%               2.676MiB / 3.848GiB   0.07%               2.11MB / 2.27MB     0B / 0B             3
    # daedd254e5ac        proxy              88.38%               9.25MiB / 3.848GiB    0.23%               133MB / 58.5MB      0B / 0B             6
```

# Chronomètre
- 2 containers: 7m30
- 4 containers: 5m12