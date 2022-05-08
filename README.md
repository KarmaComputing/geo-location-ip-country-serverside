# Geo Location IP Country Detection

Goal: As a web appliation, every request from a client 
includes the country code in the request header. Having the country code in the request header makes it easier for server side applications (in any language) to know the origin country of the request.


## Install

Ref https://github.com/maxmind/libmaxminddb#on-ubuntu-via-ppa
```
sudo apt update
sudo apt-get install -y apache2-dev apache2
sudo add-apt-repository ppa:maxmind/ppa
```

```
sudo apt update && sudo apt install libmaxminddb0 libmaxminddb-dev mmdb-bin
```

Install the apache module: https://github.com/maxmind/mod_maxminddb:
```
curl -L -O https://github.com/maxmind/mod_maxminddb/releases/download/1.2.0/mod_maxminddb-1.2.0.tar.gz
tar xvf mod_maxminddb-1.2.0.tar.gz
cd mod_maxminddb-1.2.0/
./configure
make install
a2enmod headers
systemctl restart apache2
apt install -y python3.8-venv
```

Get the geo location ip data:
```
cd
git clone https://github.com/geoacumen/geoacumen-country.git
cd /root/geoacumen-country
python3 -m venv venv
. venv/bin/activate
# Update database to latest version
python create.py # may take a while
```

## Configure apache

`vi /etc/apache2/sites-enabled/000-default.conf`:

```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        MaxMindDBEnable On
        MaxMindDBFile COUNTRY_DB /root/geoacumen-country/Geoacumen-Country.mmdb
        MaxMindDBEnv GEO_COUNTRY_CODE COUNTRY_DB/country/iso_code


        # Set the country code in the response header
        Header always set GEO-COUNTRY-CODE "%{GEO_COUNTRY_CODE}e"
        RequestHeader set Geo-Country-Code "%{GEO_COUNTRY_CODE}e"
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```
Restart apache
```
systemctl restart apache2
```

## Add a simple web page

Remove default apache page:
```
rm /var/www/html/index.html
```


# Verify

When you visit the ip address (or web address) of your server,
then you should see the a header in the responde named `GEO-COUNTRY-CODE` set to the country you're visiting from.

> Note this does not work for local networks, only public up addresses (you cannot test this locally, you need a server..)

Use chrome dev tools or `curl` to verify. e.g.:

```
$ curl -v http://<your-server-address>/
> GET / HTTP/1.1
> User-Agent: curl/7.68.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< GEO-COUNTRY-CODE: US
< Content-Type: text/html
...
```
Observe the header "GEO-COUNTRY-CODE".

# Example country-aware web app:

Create simple app (any language you prefer)

e.g. If you want to use php for some reason:
```
apt install php
systemctl restart apache2
```

`/var/www/html/index.php`

```
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Geo Location IP Country</title>
  </head>
  <body>
  <h1>You are visiting from country code: <?php echo $_SERVER['Geo-Country-Code']; ?></h1>
  </body>
</html>
```

Visit your server address and the web app will display the country you're visiting from.

# Notes

Where does the IP -> Countru data from?

- https://github.com/geoacumen/geoacumen-country
  - uses: iptoasn.com (thank you)
  - The new file format (version 2) is supported by geoacumen
- An alternative source may be via RIPE Database and other bodies

New format spec: https://maxmind.github.io/MaxMind-DB/
