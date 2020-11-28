---
title: "Ssrf Notes"
date: 2020-11-05T20:20:57+08:00
tags: 
- ssrf
- docker
categories:
draft: false
author: prontosil
authorEmoji: 👻
hideToc: false
enableToc: true
enableTocContent: false
description: 
---



记录几道SSRF题目





## babyphp

源代码如下：

```php
<?php
if(!(isset($_POST['url']))){
    show_source(__FILE__);
}
// include_once "ping.php";
if (isset($_POST['url'])) {
    $link = $_POST['url'];
    var_dump(check($link));
    if(check($link)){
        $curlobj = curl_init();
        curl_setopt($curlobj, CURLOPT_POST, 0);
        curl_setopt($curlobj,CURLOPT_URL,$link);
        curl_setopt($curlobj, CURLOPT_RETURNTRANSFER, 1);
        $result = curl_exec($curlobj);
        curl_close($curlobj);
        echo $result;
    }else{
        echo "unknown error";
    }

}

function check($url) {
    $url = parse_url($url);
    if(isset($url['port'])){    //rewrite url if port exist
        $url['path']= ':'.$url['port'].$url['path'];
    }
    if(isset($url['scheme'])){    # filter scheme
        if(strcasecmp($url['scheme'], "ftp") === 0 || strcasecmp($url['scheme'], "telnet") === 0 || strcasecmp($url['scheme'], "dict") === 0 || strcasecmp($url['scheme'], "file") === 0 || strcasecmp($url['scheme'], "ldap") === 0){
            return FALSE;
        }
    }
    $host = $url['host'];
    // var_dump($host);

    if(!preg_match('/[a-zA-Z]/', $host)){
        $ip = $host;
        if(is_inner_ip_regx($ip)){
            return FALSE;
        }
    }else{
        $ip = gethostbyname($host);
        if($ip ===$host){
            return FALSE;
        }
        if(is_inner_ip_regx($ip)){
            return FALSE;
        }
    }
    return TRUE;
}

function is_inner_ip_regx($ip){
    $pattern = "/^(127\.0\.0\.1)|(localhost)|(10\.\d{1,3}\.\d{1,3}\.\d{1,3})|(172\.((1[6-9])|(2\d)|(3[01]))\.\d{1,3}\.\d{1,3})|(192\.168\.\d{1,3}\.\d{1,3})$/";
    if(preg_match($pattern, $ip)){
        return TRUE;
    }else{
        return FALSE;
    }
}
```



