# common linux command alternatives

## ls

    echo *
    for a in *; do echo $a; done

## ps 

    grep -a . /proc/*/cmdline
    echo -n /proc/*
    for a in /proc/*/cmdline; do echo -n $a:; cat $a; echo; done

## latency under load histogram
	# flood ping troughput = 100 concurrency X 1024 bytes/s = 102 400 b/s = 100K/s =~ 1Mbps
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
	  R2=`echo $L | awk '{print $1}'` T2=`echo $LINE | awk '{print $9}'`;
	  SPEED=$((($R2+$T2-$TOT)/$I)); IN=$((($R2-$R1)/$I)); OUT=$((($T2-$T1)/$I))
	  printf "In: %12i KB/s | Out: %12i KB/s | Total: %12i KB/s\n" $(($IN/1024)) $(($OUT/1024)) $((($IN+$OUT)/1024));
	done;
	# cat > n.sh, paste the above, then: chmod +x n.sh; ./n.sh
