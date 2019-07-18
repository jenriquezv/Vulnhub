# Recon

Usando la herramienta **netdiscover**, se identificó la dirección IP **192.168.1.186** de la maquina virtual **kioptrix**.

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
