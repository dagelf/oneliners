# common linux command alternatives

And oneliners, useful for embedded systems where common commands might be missing

## ls
	# use shell expansion to list files
	echo *
	# list files in separate lines, without ls
	for a in *; do echo $a; done

## ps 

    grep -a . /proc/*/cmdline
    echo -n /proc/*
    for a in /proc/*/cmdline; do echo -n $a:; cat $a; echo; done

## latency under load histogram
	# flood ping troughput = 100 concurrency X 1024 bytes/s = 102 400 b/s = 100K/s =~ 1Mbps FD =~ 2Mbps aggregate
	for a in `seq 1 100`; do ping -c5 -q 192.168.10.1 -s $(( 1024 - 42 )) & done|grep max|cut -d\/ -f5|sort -n|cut -d\. -f1|uniq -c                
	
##  network interface throughput
	I=1 # interval in seconds  
	DEVICE=$1; OK=0; for FOUND in `grep \: /proc/net/dev | awk -F: '{print $1}'`; do if [ "$DEVICE" = $FOUND ]; then OK=1; break; fi; done
	if [ $OK -eq 0 ]; then echo "Device not found. Should be one of these:"; grep ":" /proc/net/dev | awk -F: '{print $1}' | sed s@\ @@g; exit 1; fi
	while true; do
	  L=`grep $1 /proc/net/dev | sed s/.*://`;
	  R1=`echo $L | awk '{print $1}'`; T1=`echo $L | awk '{print $9}'`; TOT=$(($R1+$T1))
	  sleep $I
	  L=`grep $1 /proc/net/dev | sed s/.*://`;
	  R2=`echo $L | awk '{print $1}'` T2=`echo $L | awk '{print $9}'`;
	  SPEED=$((($R2+$T2-$TOT)/$I)); IN=$((($R2-$R1)/$I)); OUT=$((($T2-$T1)/$I))
	  printf "In: %12i KB/s | Out: %12i KB/s | Total: %12i KB/s\n" $(($IN/1024)) $(($OUT/1024)) $((($IN+$OUT)/1024));
	done;
	# cat > n.sh, paste the above, then: chmod +x n.sh; ./n.sh

## ip scan / network scan... aka nmap without nmap and fping without fping
	# super slow serial
	for a in `seq 1 254`; do ping -c1 192.168.88.$a > /dev/null && echo $a; done

	# parallel, wrap in $() to hide forking 
	echo $(for a in `seq 1 254`; do ping -c1 192.168.88.$a > /dev/null && echo $a & done; wait)
	
	# echo to a file if you want it in separate lines, and sort the output, so it's sequential
	rm /tmp/scan; sort -n $(for a in `seq 1 254`; do ping -c1 192.168.88.$a > /dev/null && echo $a >> /tmp/scan & done; wait; echo /tmp/scan)
	
	# if cpu can't handle all parallel, just run $p at a time: add -w1 to ping to speed it up to $e/$p seconds worst
	echo $(n="192.168.1"; b=1; e=254; p=20; while [ $p -gt 0 ]; do c=0; while [ $c -le $(($p-1)) ]; do b=$(($b+1)); ping -w1 -c1 $n.$b > /dev/null && echo $b & c=$(($c+1)); done; wait; [ $(($b+$p)) -gt $e ] && p=$(($e-$b)); done;)
	
	# same, but lists in sequence, in separate lines, with full ip addresses
	rm /tmp/scan; sort -n -t. -k4 $(n="192.168.1"; b=1; e=254; p=20; while [ $p -gt 0 ]; do c=0; while [ $c -le $(($p-1)) ]; do b=$(($b+1)); ping -w1 -c1 $n.$b > /dev/null && echo $n.$b >> /tmp/scan & c=$(($c+1)) ; done; wait; [ $(($b+$p)) -gt $e ] && p=$(($e-$b)); done; echo /tmp/scan)
		
## port map
	nc -zv 192.168.88.1 80-10000

## benchmarks

	for a in 5 10 200 500; do echo -n -c$a:; ab -q -n $(( $a*2 )) -c $a http://172.17.0.4/sample-page/ |grep Req; done

## docker
	curl -sL https://get.docker.com | sh

## import sql and add user

	#!/bin/sh

	db=mariadb
	app=app1

	db_rootpw=...

	db_name=app1
	db_user=app1
	db_password=...
	db_sql=app1.sql.gz

	my=mysql; which $my || my="docker exec -i $db mysql"

	docker inspect $db -f '{{range .}}{{end}}' || docker start $db || docker run -d --name $db -e MYSQL_ROOT_PASSWORD=$db_rootpw mariadb
	ip_db=`docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $db`

	echo This will drop $db_name in $db and import it from $db_sql. All data will be LOST! Enter to continue!
	read $wait

	echo "echo \"DROP DATABASE IF EXISTS $db_name; CREATE DATABASE $db_name; USE mysql; GRANT ALL PRIVILEGES ON $db_name.* TO '$db_user'@'%' IDENTIFIED BY '$db_password';FLUSH PRIVILEGES\"|$my -h $ip_db -u root --password=$db_rootpw" # show command
	echo "DROP DATABASE IF EXISTS $db_name; CREATE DATABASE $db_name; USE mysql; GRANT ALL PRIVILEGES ON $db_name.* TO '$db_user'@'%' IDENTIFIED BY '$db_password';FLUSH PRIVILEGES"|$my -h $ip_db -u root --password=$db_rootpw

	#db_user=root; db_password=$db_rootpw; # if special permissions required for initial import, fixme: else fix permissions
	[ -z "$db_sql" ] || (
	echo "zcat $db_sql|$my -h $ip_db -u $db_user --password=$db_password $db_name" # show command
	zcat $db_sql|$my -h $ip_db -u $db_user --password=$db_password $db_name
	);

## git
	# use a bare repo on a master (origin) server (why?)
	vi .git/config
	git init --bare
	git pull origin otherbranch
	https://stackoverflow.com/questions/10312521/how-to-fetch-all-git-branches
	
	git clone url
	git ls-remote
	git branch
	git checkout mybranch
	git push --set-upstream origin mybranch
	
	# create gita push and gita pull and gita clone to sync all the branches? 
	
	# create branches only when working on things that might take longer than 5 mins
	
	gitk

## docker

	# find docker path, useful for old dockers where you need to copy files in
	a=anchor; z=`pwgen 6 1`; docker exec $a touch /$z;sudo find /var/lib/docker/|grep $z|sed s@$z@@;docker exec $a rm /$z

## dnsmasq

	# if you're getting "dnsmasq: faile to create listenting socket for port 53: Address already in use"
	# on Ubuntu or Debian, see the dnsmasq section in https://github.com/dagelf/pi
	# run a dhcp server in foreground
	#ifconfig eth0 192.168.1.1 || ip addr add 192.168.1.1/24 dev eth0 
	dnsmasq -d -i eth0 -F 192.168.1.10,192.168.1.199
	
	# and share tftp file

# udhcpc
	# try to get a new lease... where's the log/cache?
	udhcpc -f -r192.168.9.43 -i eth0 -R -q
	
# webserver oneliners (run a web server in the current path)

	while true ; do nc -l 80 < index.html ; done
	python -m SimpleHTTPServer 8000
	php -S 127.0.0.1:8000
	php artisan serve --host=127.0.0.1 --port=8000
	python3 -m http.server 8000
	twistd -n web -p 8000 --path .
	python -c 'from twisted.web.server import Site; from twisted.web.static import File; from twisted.internet import reactor; reactor.listenTCP(8000, Site(File("."))); reactor.run()'
	ruby -rwebrick -e'WEBrick::HTTPServer.new(:Port => 8000, :DocumentRoot => Dir.pwd).start'
	ruby -run -ehttpd . -p8000
	gem install adsf; adsf -p 8000
	cpan HTTP::Server::Brick  
	perl -MHTTP::Server::Brick -e '$s=HTTP::Server::Brick->new(port=>8000); $s->mount("/"=>{path=>"."}); $s->start'
	npm install -g http-server 
	http-server -p 8000
	busybox httpd -f -p 8000
	webfsd -F -p 8000
	pip install djangothis; djangothis
	# more at https://gist.github.com/willurd/5720255
	
# display messages (active or executable readme)
	#!/bin/sh
	tail -n$(($(cat $0|wc -l)-2)) $0;exit
	# everything from here on will be displayed
	# todo: make one that displays with substitutions, and/or a templated version
	# this is useful for scripts that are not generic enough to just run with parameters
	# and where some thinking is required
	
	

