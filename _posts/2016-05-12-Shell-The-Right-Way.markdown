---
layout: post
title: "Writing Shell. The Right Way!"
date: 2016-05-12 08:18:29
categories: Shell
---

Shell is simple but not esay. If you don't wanto to leave spaghetti to others, you have to be careful to this tricky things.



It's not a good idea to print the log to STD_IN, so we need archive these log into a log file.

{% highlight xml %}
log()
{
    local level=$1
    local msg=$2
    local time_str=`date +'%Y-%m-%d %H:%M:%S'`
    echo "$time_str $level - $msg" >> $LOG_PATH/$LOG_FILE
}

{% endhighlight %}

Log file will become very large if you don't carefully treat it. It's wise to archive the log by the date.

{% highlight xml %}

init_log_file()
{
    if [ ! -d $LOG_PATH ]; then
        mkdir -p $LOG_PATH
    fi

    local date_str=`date +%Y-%m-%d`
    LOG_FILE=postgres_monitor.log.$date_str

    if [ ! -f $LOG_PATH/$LOG_FILE ]; then
        touch $LOG_PATH/$LOG_FILE
    fi
}

{% endhighlight %}
