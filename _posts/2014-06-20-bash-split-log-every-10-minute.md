---
layout: post
tagline: "Supporting tagline"
tags : [bash]
title : bash - split log every 10 minute
---
{% include JB/setup %}

We have a system which logs all http request, including timestamp, client ip, server ip, http status and url. 

We have three machine collecting these logs, and deployed a job on storm to merge them all and do completely analysis.

    machine A logs \
    machine B logs -> flume -> kafaka -> storm
    machine C logs /

As storm is rarely used in our company, they had not provide a user friendly web interface to submit the storm job. So, i write a script do a simple analyse.

The log file are rotate every hour, and size 1G ~ 2G.

	$ls -l -h
	-rw-rw-r--+ 1 root root 1006M Jun 20 10:30 
	-rw-rw-r--+ 1 root root 1.4G Jun 20 08:59 monitor.log.20140620-08
	-rw-rw-r--+ 1 root root 1.9G Jun 20 09:59 monitor.log.20140620-09

I want to count the http status every 10 minutes, and send me a report if there are too much http 404 error.

	http status in last ten minutes:

	 103607 304
	  53745 410
	  50281 404
	  36587 302
	  22131 301
	   6299 201
	    426 502
	    341 202
	    300 504
	    175 400
	    148 401
	    125 405
	     69 100
	     53 403
	     39 206
	     37 500
	      1 503

	Top ten 404 urls:

	   1005 url1
	    877 url2
	    546 url3
	    496 url4
	    352 url5
	    305 url6
	    243 url7
	    226 url8
	    222 url9
	    186 url10

The script:
{% highlight bash %}

#!/bin/bash

# Author : hailinzeng
# Usage : add in crontab */10 * * * * /opt/monitor/report-404.sh >> /opt/monitor/report-404.log 2>&1

time_10min_ago=`date --date "-10min" "+%Y-%m-%d %H:%M"`
filename_from_time=`echo $time_10min_ago |sed "s/ /./g" |sed "s/:/-/g"`

#dir
root=/opt/monitor

outputlog="$root/$filename_from_time.404.log"
inputfile="$root/monitor.log"
periodlog="$root/report-404.period.log"

#reset filename if rotate
minute_countby_10=`echo $time_10min_ago |cut -c15-15`

if [[ $minute = "0" ]]; then
    filename=`echo $time_10min_ago |cut -c1-13 |sed "s/-//g" |sed "s/ /-/g"`
    inputfile="$root/$filename"
fi

echo "$time_10min_ago"

#wait write log finish, and rotate finish
sleep 10

#logs which time lay between [x, x+10)
time_10min_ago_wo_rmb=`echo $time_10min_ago |cut -c1-15`

grep "^$time_10min_ago_wo_rmb" $inputfile |grep -v "\"Status\":200" > $periodlog

#count http state
echo -e "\nhttp status in last ten minutes:\n" > $outputlog

cat $periodlog |sh $root/timecode.sh |cut -c15-16,24- |awk '{print $2}' |sort |uniq -c |sort -k1 -nr >> $outputlog

#404
count404=`grep -P "\d+ 404" $outputlog |awk '{print $1}'`

if [[ "$count404" -gt "50000" ]]; then
    #list top 404 url
    echo -e "\nTop ten 404 url:\n" >> $outputlog

    cat $periodlog |grep "\"Status\":404" |sh $root/url.sh |sh $root/urldecode.sh |sh $root/urlnoparm.sh |sort |uniq -c |sort -nr -k 1 |head -10 >> $outputlog

    #send mail
    #cat $outputlog |/usr/bin/mutt -s "HTTP 404" xxx
fi

rm -rf $outputlog

{% endhighlight %}
