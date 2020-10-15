# Test Redis v4 and v6 cluster

### Start it up

```
docker-compose up
```

### Create cluster of v4 instances

```
docker-compose exec master-6 redis-cli --cluster create \
	$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis46_master-{0,1,2}_1 | xargs -I% echo %:6379)
```

### Join in the v6 instance

```
docker-compose exec master-6 redis-cli --cluster add-node  \
  $(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis46_master-6_1):6379 \
  $(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis46_master-0_1):6379
```

### Check

```
docker-compose exec master-6 redis-cli cluster nodes                                                                                                                            <<<
```

```
7dd63328e6c4e27254cf4e4c944a6ffa955396d3 172.20.0.2:6379@16379 master - 0 1602749322043 2 connected 5461-10922
4e206db5bf16fa336bb0682c122c860b34ca31ec 172.20.0.5:6379@16379 myself,master - 0 1602749322000 0 connected
49287f1bcc757c821d324ca02bf34daf49ff193b 172.20.0.4:6379@16379 master - 0 1602749323000 1 connected 0-5460
d1ad28d88d6520065b070b7092ca68d4d6e8a4d6 172.20.0.3:6379@16379 master - 0 1602749323150 3 connected 10923-16383
```

### Assign some slots to the v6

```
docker-compose exec master-6 redis-cli --cluster rebalance --cluster-use-empty-masters \
  $(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis46_master-0_1):6379
```

```
docker-compose exec master-6 redis-cli cluster nodes
7dd63328e6c4e27254cf4e4c944a6ffa955396d3 172.20.0.2:6379@16379 master - 0 1602749865728 2 connected 6827-10922
4e206db5bf16fa336bb0682c122c860b34ca31ec 172.20.0.5:6379@16379 myself,master - 0 1602749866000 4 connected 0-1364 5461-6826 10923-12287
49287f1bcc757c821d324ca02bf34daf49ff193b 172.20.0.4:6379@16379 master - 0 1602749865223 1 connected 1365-5460
d1ad28d88d6520065b070b7092ca68d4d6e8a4d6 172.20.0.3:6379@16379 master - 0 1602749866229 3 connected 12288-16383
```
