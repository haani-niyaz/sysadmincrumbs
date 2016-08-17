## VirtualHosts


### Access Multiple Apss from Reverse Proxy on a Single Host

```

booking (app 1)
monitoring (app 2)
   |
   [Reverse Proxy]
   				 |
   				 host
   				 	\ booking
   				 	\ monitoring
```


#### Reverse Proxy:

```
/**
 * Reverse Proxy
 * 
 */

#
# Use name-based virtual hosting.
#
NameVirtualHost *:80


# /etc/httpd/conf.d/virtualhost-monitor.conf
<VirtualHost *:80>
	ServerName monitor.com
	ErrorLog logs/monitor.com-error_log
	TransferLog logs/monitor.com-access_log
	ProxyPreserveHost On
	ProxyPass / http://192.168.33.11/
</VirtualHost>


# /etc/httpd/conf.d/virtualhost-booking.conf
<VirtualHost *:80>
	ServerName booking.com
	ErrorLog logs/booking.com-error_log
	TransferLog logs/booking.com-access_log
	ProxyPreserveHost On
	ProxyPass / http://192.168.33.11/
</VirtualHost>

```


#### Host:

```
/**
 * Host
 * 
 */


#
# Use name-based virtual hosting.
#
NameVirtualHost *:80


# /etc/httpd/conf.d/virtualhost-monitor.conf
 <VirtualHost *:80>
	ServerName monitor.com
	ErrorLog logs/monitor.com-error_log
	TransferLog logs/monitor.com-access_log
	DocumentRoot /var/www/monitor
</VirtualHost>


# /etc/httpd/conf.d/virtualhost-booking.conf
<VirtualHost *:80>
	ServerName booking.com
	ErrorLog logs/booking.com-error_log
	TransferLog logs/booking.com-access_log
	DocumentRoot /var/www/booking
</VirtualHost>
```


#### Errors


```
[root@localhost conf.d]# sudo /etc/init.d/httpd restart
Stopping httpd:                                            [  OK  ]
Starting httpd: [Wed Aug 17 10:39:07 2016] [warn] _default_ VirtualHost overlap on port 80, the first has precedence
                                                           [  OK  ]
```


You will get the above warning, which will result in only the first site being served no matter which domain you hit. This is why you need to set `NameVirtualHost *:80` in `/etc/httpd/conf/httpd/conf`. This tells your server to serve different virtual hosts based on the doman names used to reach it.

Ensure you do a restart and clean your browser cache to avoid any issues.
