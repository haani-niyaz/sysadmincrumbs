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

# /etc/httpd/conf/httpd.conf

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
