# HAProxy Sandbox

The purpose of this repo is to have a minimalistic example of HAProxy rules (which you
can easily modify) and instructions on how to create a development server with a few
servieces (WordPress and a minimalistic API) so that you'll be able to live test these
HAProxy rules.

Stack:
- Ubuntu (because it has faster dev cycle compare to NixOS)
- HAProxy
- WordPress Blog
- Gin api

## Set Up Development Environment

First you'll need to create a new server. This can be on AWS, Linode, DigitalClout, etc.

### Install Requirements

```
$ sudo apt update && sudo apt upgrade -y
$ sudo apt install haproxy
$ sudo apt install golang-go
```

HAProxy configuration located at `/etc/haproxy/haproxy.cfg`.

Optional install zsh and micro:
```
$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
$ sudo apt install micro
```

Transfer `/api` folder to the server, build it (`go build`) and run it (`./hsapi`)

You might also need to disable the firewall (`sudo ufw disable`) or allow traffic via
port 80.

## Testing

We'll use [`hay`](https://github.com/rakyll/hey) to test things like rate limiting.

Example (rate limit in action):
```
$ hey -n 100 -c 3 http://<IP>
Summary:
  Total:	0.9186 secs
  Slowest:	0.0909 secs
  Fastest:	0.0202 secs
  Average:	0.0272 secs
  Requests/sec:	107.7760
  
  Total data:	9743 bytes
  Size/request:	98 bytes

Response time histogram:
  0.020 [1]	|■
  0.027 [70]	|■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  0.034 [22]	|■■■■■■■■■■■■■
  0.041 [2]	|■
  0.048 [1]	|■
  0.056 [0]	|
  0.063 [0]	|
  0.070 [0]	|
  0.077 [0]	|
  0.084 [0]	|
  0.091 [3]	|■■


Latency distribution:
  10% in 0.0211 secs
  25% in 0.0229 secs
  50% in 0.0244 secs
  75% in 0.0279 secs
  90% in 0.0321 secs
  95% in 0.0425 secs
  0% in 0.0000 secs

Details (average, fastest, slowest):
  DNS+dialup:	0.0016 secs, 0.0202 secs, 0.0909 secs
  DNS-lookup:	0.0000 secs, 0.0000 secs, 0.0000 secs
  req write:	0.0000 secs, 0.0000 secs, 0.0003 secs
  resp wait:	0.0254 secs, 0.0201 secs, 0.0424 secs
  resp read:	0.0001 secs, 0.0000 secs, 0.0010 secs

Status code distribution:
  [200]	20 responses
  [429]	79 responses

```

You can see these results in the HAProxy log. This file is usually located in
`/var/log/haproxy.log`. Check your `/etc/rsyslog.d` settings.


```
...
Sep 23 10:05:05 karantan haproxy[2149]: 93.103.124.110:52098 [23/Sep/2022:10:05:05.497] web api/server1 0/0/0/0/0 200 151 - - ---- 3/3/0/0/0 0/0 {70.34.213.|hey/0.0.1} "GET / HTTP/1.1"
Sep 23 10:05:05 karantan haproxy[2149]: 93.103.124.110:52097 [23/Sep/2022:10:05:05.502] web api/server1 0/0/0/0/0 200 151 - - ---- 3/3/0/0/0 0/0 {70.34.213.|hey/0.0.1} "GET / HTTP/1.1"
Sep 23 10:05:05 karantan haproxy[2149]: 93.103.124.110:52096 [23/Sep/2022:10:05:05.544] web web/<NOSRV> 0/-1/-1/-1/0 429 225 - - PR-- 3/3/0/0/0 0/0 {70.34.213.|hey/0.0.1} "GET / HTTP/1.1"
Sep 23 10:05:05 karantan haproxy[2149]: 93.103.124.110:52097 [23/Sep/2022:10:05:05.544] web api/server1 0/0/0/1/1 200 151 - - ---- 3/3/0/0/0 0/0 {70.34.213.|hey/0.0.1} "GET / HTTP/1.1"
Sep 23 10:05:05 karantan haproxy[2149]: 93.103.124.110:52098 [23/Sep/2022:10:05:05.548] web web/<NOSRV> 0/-1/-1/-1/0 429 225 - - PR-- 3/3/0/0/0 0/0 {70.34.213.|hey/0.0.1} "GET / HTTP/1.1"
Sep 23 10:05:05 karantan haproxy[2149]: 93.103.124.110:52097 [23/Sep/2022:10:05:05.590] web web/<NOSRV> 0/-1/-1/-1/0 429 225 - - PR-- 3/3/0/0/0 0/0 {70.34.213.|hey/0.0.1} "GET / HTTP/1.1"
Sep 23 10:05:05 karantan haproxy[2149]: 93.103.124.110:52096 [23/Sep/2022:10:05:05.591] web web/<NOSRV> 0/-1/-1/-1/0 429 225 - - PR-- 3/3/0/0/0 0/0 {70.34.213.|hey/0.0.1} "GET / HTTP/1.1"
...
```

## Resources

- [HAProxy Rate Limiting](https://www.haproxy.com/blog/four-examples-of-haproxy-rate-limiting/)
- [HAProxy Configuration](https://www.haproxy.com/documentation/hapee/latest/configuration/config-sections/overview/)
- [HAProxy ACLs](https://www.haproxy.com/documentation/hapee/latest/configuration/acls/overview/)
- [HAProxy Stick Tables](https://www.haproxy.com/blog/introduction-to-haproxy-stick-tables/)
