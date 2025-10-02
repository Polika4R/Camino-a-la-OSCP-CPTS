ina
**Answer the question(s) below to complete this Section and earn cubes!
Target(s): 94.237.54.192:32577
Life Left: 87 minute(s)
vHosts needed for these questions:
- `inlanefreight.htb**

**What is the IANA ID of the registrar of the inlanefreight.com domain?**

Añado al /etc/hosts:
```
94.237.54.192 inlanefreight.htb
```

Y realizo un *whois* filtrando por *IANA*:
```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-zr9xwsub5m]─[~]
└──╼ [★]$ whois inlanefreight.com | grep "IANA"
   Registrar IANA ID: 468
Registrar IANA ID: 468
```

Siendo la respuesta: 468.


**What http server software is powering the inlanefreight.htb site on the target system? Respond with the name of the software, not the version, e.g., Apache.**

Realizo un:

```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-zr9xwsub5m]─[~]
└──╼ [★]$ curl -I inlanefreight.htb:32577
HTTP/1.1 200 OK
Server: nginx/1.26.1
Date: Sat, 23 Aug 2025 17:26:29 GMT
Content-Type: text/html
Content-Length: 120
Last-Modified: Thu, 01 Aug 2024 09:35:23 GMT
Connection: keep-alive
ETag: "66ab56db-78"
Accept-Ranges: bytes

```

Y observo que la respuesta es *nginx*.


**3. What is the API key in the hidden admin directory that you have discovered on the target system?**
Realizo una búsqueda con wfuzz y encuentro: 

```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-zr9xwsub5m]─[~]
└──╼ [★]$ wfuzz --hh=120 -t 50 -c -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.inlanefreight.htb' http://94.237.54.192:32577
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://94.237.54.192:32577/
Total requests: 114441

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                
=====================================================================

000107876:   200        1 L      4 W        104 Ch      "web1337"                                                                                                              

Total time: 0
Processed Requests: 114441
Filtered Requests: 114440
Requests/sec.: 0

```


Añado al /etc/hosts:

```
94.237.54.192 web1337.inlanefreight.htb
```

Y realizo una nueva búsqueda pero ahora de archivos, con:
```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-zr9xwsub5m]─[~]
└──╼ [★]$ wfuzz --hh=153 -t 50 -c -w /usr/share/wordlists/dirb/common.txt -H 'Host: web1337.inlanefreight.htb' http://94.237.54.192:32577/FUZZ
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://94.237.54.192:32577/FUZZ
Total requests: 4614

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                
=====================================================================

000000001:   200        1 L      4 W        104 Ch      "http://94.237.54.192:32577/"                                                                                          
000002020:   200        1 L      4 W        104 Ch      "index.html"                                                                                                           
000003436:   200        5 L      10 W       99 Ch       "robots.txt"                                                                                                          
```

Accedo por el navegador a:
http://web1337.inlanefreight.htb:32577/robots.txt

Y observo:
```
User-agent: *
Allow: /index.html
Allow: /index-2.html
Allow: /index-3.html
Disallow: /admin_h1dd3n
```

Accedo pues a:
http://web1337.inlanefreight.htb:32577/admin_h1dd3n

Y pego la flag encontrada: 
```
Welcome to web1337 admin site
The admin panel is currently under maintenance, but the API is still accessible with the key e963d863ee0e82ba7080fbf558ca0d3f[[10 - Fingerprinting]]
```

**After crawling the inlanefreight.htb domain on the target system, what is the email address you have found? Respond with the full email, e.g., mail@inlanefreight.htb**

Realizo un escaneo de VHOST y observo:
```
┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-zr9xwsub5m]─[~]
└──╼ [★]$ wfuzz --hh=120 -t 200 -c -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.web1337.inlanefreight.htb' http://94.237.54.192:32577
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://94.237.54.192:32577/
Total requests: 114441

=====================================================================
ID           Response   Lines    Word       Chars       Payload       
=====================================================================

000000019:   200        0 L      5 W        123 Ch      "dev"   
```

Lo añado al /etc/hosts con:
```
94.237.54.192 inlanefreight.htb web1337.inlanefreight.htb dev.web1337.inlanefreight.htb

```

INstalo ReconSpyder:

```
 >pip3 install scrapy
 >wget -O ReconSPider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
 >unzip ReconSPider.zip
 
```

Ejecuto el script:
```
python3 ReconSpider.py http://dev.web1337.inlanefreight.htb:32577
```

 Y hago un cat al resultado mostrado:
 ```
 cat results.json
 ```

Con un *cat*:
```
}┌─[eu-academy-3]─[10.10.15.85]─[htb-ac-1876550@htb-zr9xwsub5m]─[~]
└──╼ [★]$ cat results.json | grep "@"
        "1337testing@inlanefreight.htb"
```

Veo que la respuesta es:  1337testing@inlanefreight.htb

**What is the API key the inlanefreight.htb developers will be changing too?**


Sobre la salida anterior observo: 
```
"<!-- Remember to change the API Key to ba988b835be4aa97d068941dc852ff33 --->"
```

Respuesta: ba988b835be4aa97d068941dc852ff33

