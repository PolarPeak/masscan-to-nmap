# masscan-to-nmap

## Why?
Masscan it's much faster than nmap. But NSE scripts are amazing. I wrote these scripts to scan open ports  in large bug bounty scopes very fast with masscan and then with nmap with the NSEs I desire in shorter time. You also should know that you can't provide to nmap a specific port for an especific host in a list (check https://github.com/nmap/nmap/issues/1217). Maybe it could be helpful to some folks.

## Steps
* First I recommend you to read masscan and nmap usage documentation.
* Create a text file with all the network segments addresses you want. Domains and ASNs are a good starting point. In this example, I will name it "ip_list.txt"
* Get a list of ports you want to scan. I added my list in "ports.txt". But now let' s say it's only 80 and 443.
* Launch a masscan scan and save the report in xml format with  ```masscan -iL ip_list.txt -e eth0 -oX report_masscan.xml -p80,443 --max-rate 100000```
* Now we will generate a list with ONLY the open ports of each scanned ip address.  Parse the xml and pipe it to a file with ```python3 masscantolist.py report_masscan.xml > ip_ports.txt  ```
* Now we will generate the nmap scans, one for each port (so you won't scan ports that you already know are closed). Run ```python3 gennmapscans.py ip_ports.txt "MyNmapReports""``` (naming my scans allow me to be more organized)
* A new folder will be created named "GNMAPSCAN_MyNmapReports_YY-MM-DD_HH-MM-SS" (includes the report name we gave to gennmapscans.py and a timestamp). The folder will contain:
  *  One file gscan_[PORT_NUMBER].list per each port. You can customize each file according to your needs (for example, run specific NSEs for certain ports or changing scans order). NOTE: if many files are created check the size of them, some hosts usually return that all the ports are open, generating small files with trash.
  *  A file "runme.sh".
* When you have finished editing the runme.sh, launch it to start the scans.
* The default behaviour will create a report in both standard and xml format.
* Wait all the scans until they are finished. Now you have the reports.

## Extra
* You can modify the resulting nmap arguments editing the file "gennmapscans.py" (line 5, variable NMAP_CMD).
* I use to navigate the info from the reports with grep. For example ```grep -nH GNMAPSCAN_MyNmapReports_YY-MM-DD_HH-MM-SS/*.nmap "Apache Banner"```.
* You may run many nmaps at the same time. But remember that nmap use hosts and ports randomization, so I wouldn't recommend if you want to avoid WAFs and firewalls.
* There is an extra script "domainsfromnmapxml.py". I use it to extract the domain name from the "ssl-cert" from te HTTPS ports and get domain names. It may be helpful for other people too.

