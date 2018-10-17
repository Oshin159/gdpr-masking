Install mod_geoip2 module
sudo apt-get install libapache2-mod-geoip
enable headers module in apache
Add the following lines in apache conf file
<IfModule mod_geoip.c>
GeoIPEnable On
GeoIPEnableUTF8 On
GeoIPDBFile /etc/apache2/maxmind/mmcountry.dat MemoryCache
GeoIPOutput Env
GeoIPScanProxyHeaders On
GeoIPUseLastXForwardedForIP On
GeoIPScanProxyHeaderField X-Client-IP
<IfModule mod_headers.c>
<If "reqenv('GEOIP_COUNTRY_CODE') in { 'EU','AT', 'BE', 'BG', 'CY', 'CZ', 'DE', 'DK', 'EE', 'ES', 'FI', 'FR', 'GB', 'GR', 'HR', 'HU', 'IE', 'IS', 'IT', 'LI', 'LT', 'LU', 'LV', 'MT', 'NL','NO', 'PL', 'PT', 'RO', 'SE', 'SI', 'SK' }">
#In case of EU traffic execute the following
RequestHeader setifempty IS-EU 1
SetEnvIf X-Client-IP "^(\d+.\d+.\d+).\d+$" XCI-HideIP=$1.0
SetEnvIfExpr "req_novary('X-Forwarded-For') =~ /(.*)/" XFF-HideIP="-"

</If>
<Else>
RequestHeader setifempty IS-EU 0
SetEnvIfExpr "req_novary('X-Client-IP') =~ /(.*)/" XCI-HideIP=$1
SetEnvIfExpr "req_novary('X-Forwarded-For') =~ /(.*)/" XFF-HideIP=$1
</Else>
</IfModule>
</IfModule>

Now u can either use maxmind database
http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz
In this case u have to replace mmcountry.dat to GeoIP.dat

Now in case we wish to use netacuity ip2country,some amount of processing needs to be done there

Download netacuity database:
ex- ip2co-data-06-2018.csv


run the following python ip2co_convert.py

python convert.py ip2co-data-06-2018.csv maxmind.csv
ip2co_convert.py u can remove the if statement as it takes more time and run
sed -i '/\*\*/d' maxmind.csv
This has to be done
sed -i 's/UK/GB/g' maxmind.csv
after that u need to convert the generated csv to .dat file
https://github.com/mteodoro/mmutils/blob/master/csv2dat.py
python csv2dat.py -w mmcountry.dat mmcountry maxmind.csv
