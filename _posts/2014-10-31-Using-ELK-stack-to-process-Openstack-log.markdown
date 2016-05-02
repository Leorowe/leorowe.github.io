---
layout: post
title: "Using ELK stack to process Openstack log"
date: 2014-10-31 22:51:49
categories: IT
---

## introduction to ELK

ELK is the abbreviation of Elasticsearch, Logstash and Kibana.
As you can see, Here using 4 complementary applications, the role of each one being :

 - Logstash-forwarder: transfer data log to logstash
 - Logstash: extract info from log. make unstructured log into meaningful and searchable. 
 - ElascticSearch: store the data that logstash processed and provide full-text index
 - Kibana: provide a friendly web console for user to interact with elasticsearch.

## Logstash-forwarder 

### Build from source 
  1. Install [go](https://golang.org/doc/install) environment
  2. Compile [logstash-forwarder](https://github.com/elasticsearch/logstash-forwarder#building-it)
  3. Generating an ssl certificate
	 3.1. modify openssl.cnf
	 Edit your /etc/ssl/openssl.cnf on the logstash host - add `subjectAltName = IP:[Your Server IP]` in **[v3_ca]** section.
	 3.2. create the certificate and key
     `openssl req -x509 -batch -nodes -newkey rsa:2048 -keyout logstash-forwarder.key -out logstash-forwarder.crt -days 365`
     3.3. copy the crt and key file to both hosts.
  4. Edit [configuration file](https://github.com/elasticsearch/logstash-forwarder#configuring)

		{
		      "network": {
		        "servers": [ "Your IP:2329" ],
		        "ssl ca": "/root/evyzzcm/logstash-forwarder.crt",
		        "timeout": 15
		      },

		      "files": [
		        {
		          "paths": [ "/var/log/nova/*.log" ],
		          "fields": { "type": "openstack","component":"nova" }
		        },
		        {
		          "paths":[ "/var/log/neutron/*.log"],
		          "fields": { "type": "openstack","component":"neutron" }
		        }
		      ]
		}

  In this configuration file,  Logstash-forwarder will monitor the variations(tail mechanism) of all log file under directory of `/var/log/nova  ` and `/var/log/neutron`.This means logstash-forwarder will only transfer 
 new appended log to Logstash. If you want the old log also be conveyed, just copy certain log file to a new temp log will work. For example, `cp old.log ./temp.log` . Logstash-forwarder will recognize the temp.log exist and transfer it to Logstash.
 5.  Boot up Logstash-forwarder
      `logstash-forwarder -config logstash-forwarder.conf`

### Install from RPM ###
If you have build a native packages of logstash-forwarder, you could use `yum` to install it directly.

 1. Upload rpm to the machine
 2. Use `yum` to install rpm
	 `yum install logstash-forwarder-0.3.1-1.x86_64.rpm`
	 Packages will be installed to /opt/logstash-forwarder.
 3. Place `logstash-forwarder.conf` and `logstash-forwarder.crt` under **/opt/logstash-forwarder/** folder.
	 If you want to modify crt file and configuration file, modify the file in **/etc/init.d/logstash-forwarder**
 4. Add logstash-forwarder into boot up service
	 `chkconfig -add logstash-forwarder`
 5. Boot up Logstash-forwarder
	 `service logstash-forwarder start` 
 
## ELK stack
Here we use docker to run ELK stack.You can check this [post](http://datapsyche.wordpress.com/2014/07/30/docker-app-tutorial-creating-a-docker-container-for-elk-elasticsearch-logstash-kibana/) to see how to create a docker container for ELK.

`docker pull cyberabis/docker-elkauto` 

`docker run -d -p 80:80 -p 3333:3333 -p 3334:3334 -p 9200:9200 cyberabis/docker-elkauto /elk_start.sh`

### Logstash workflow

	|---------|         |----------|       |----------|
	|  Input  |  ------>|  Filter  |------>|  Output  |
	|---------|         |----------|       |----------|

Inputs, Outputs and Filters are at the heart of the Logstash configuration. Logstash process pipeline with 3 stages: inputs -> filters -> outputs. In a word, Inputs *generate* events, filters *modify* them, outputs *ship* them elsewhere. If you wan to know the detail about the life of an event, check this [document](http://logstash.net/docs/1.4.2/life-of-an-event).

The following is the logstash configuration file:

	input {
	  lumberjack {
	    port => 2329
	    ssl_certificate => "/{your crt location}/logstash-forwarder.crt"
	    ssl_key => "/{your key location}/logstash-forwarder.key"
	  }
	}
	filter {
	  if [type] == 'openstack-glance'{
	    multiline {
	      negate => true
	      pattern => "^%{TIMESTAMP_ISO8601} "
	      what => "previous"
	      stream_identity => "%{host}.%{filename}"
	    }
	    multiline {
	      negate => false
	      pattern => "^%{TIMESTAMP_ISO8601}%{SPACE}%{NUMBER}?%{SPACE}?TRACE"
	      what => "previous"
	      stream_identity => "%{host}.%{filename}"
	    }
	    grok{
	         patterns_dir => "/logstash/patterns"
	         match=>[ "message","%{TIMESTAMP_ISO8601:timestamp} %{NUMBER:response} %{AUDITLOGLEVEL:level} %{NOTSPACE:module} \[%{GREEDYDATA:program}\] %{GREEDYDATA:content}"]
	       }
	       date{
	        match=>['timestamp','YYYY-MM-dd HH:mm:ss.SSS']
	       }
	   }
	}
	output { elasticsearch { host => localhost } }

 - lumberjack
     alias logstash-forwarder,which accept input from logstash-forwarder.
 - multiline 
	In default logstash process one line one time. However, some exception log are multiline.Here [multiline](http://logstash.net/docs/1.4.2/filters/multiline) are used to collapse multiline messages into ones. 
 - date
   Logstash only support two datatype, which is Integer and String respectively. In order to draw data into time histogram, timestamp should be convert into date type in Elasticsearch and this is role that [date](http://logstash.net/docs/1.4.2/filters/date) play in.
 - grok
	The key point to extract information from log is to write correct regular expression that match log format.
[Grok](http://grokdebug.herokuapp.com/patterns) provide many pre-defined regular expression to reduce your load.You can test your Grok expression in this [online debugger](http://grokdebug.herokuapp.com/patterns) at first.
  For example, the following grok expression will match OpenStack log format.

`%{TIMESTAMP_ISO8601:timestamp} %{NUMBER:response} %{AUDITLOGLEVEL:level} %{NOTSPACE:module} \[%{GREEDYDATA:program}\] %{GREEDYDATA:content}`

  Here the AUDITLOGLEVEL is the self-defined tag. so create a new file in /{logstash-dir}/patterns/ . And add following content.

  `AUDITLOGLEVEL ([C|c]ritical|CRITICAL[A|a]udit|AUDIT|[D|d]ebug|DEBUG|[N|n]otice|NOTICE|[I|i]nfo|INFO|[W|w]arn?(?:ing)?|WARN?(?:ING)?|[E|e]rr?(?:or)?|ERR?(?:OR)?|[C|c]rit?(?:ical)?|CRIT?(?:ICAL)?|[F|f]atal|FATAL|[S|s]evere|SEVERE)`
  
  Now,let's see what will be extract from this sample OpenStack log:
	
  `2014-10-17 12:59:43.939 7399 WARNING keystoneclient.middleware.auth_token [-] Configuring auth_uri to point to the public identity endpoint is required; clients may not be able to authenticate against an admin endpoint`
	
  The following fields will be extracted for you iptables logs: 
  
  - timestamp: 2014-10-17 12:59:43.939
  - response: 7399 
  - level: WARNING
  - module: keystoneclient.middleware.auth_token
  - program: \-
  - content: Configuring auth_uri to point to the public identity endpoint is required; clients may not be able to authenticate against an admin endpoint

### Configure Kibana

#### change the [elasticsearch port](http://www.elasticsearch.org/guide/en/kibana/current/using-kibana-for-the-first-time.html#using-kibana-for-the-first-time)
The Kibana configuration file is in `/usr/share/nginx/html/config.js`. Find the filed `elasticsearch` and modify the port to what you want. 

#### 10 minute[ walk through](http://www.elasticsearch.org/guide/en/kibana/current/using-kibana-for-the-first-time.html#using-kibana-for-the-first-time)

### Administer ElasticSearch

 - debug what data have been stored in ElasticSearch
   `curl 'http://localhost:9200/_search?pretty'`
 - clean up all data
   `curl -XDELETE 'http://localhost:9200/*/'`
 - Persistent log data to host storage.
   As we know, once the docker is power off, the data will fade away otherwise you explicitly store it to host storage. So when we boot up elk image, we mount the elasticsearch data file folder to host file system.
   `docker run -d -p 80:80 -p 3333:3333 -p 3334:3334 -p 9200:9200 -v /{host-location}/docker-elasticsearch:/elasticsearch/data cyberabis/docker-elkauto /elk_start.sh `
 - Do free text search  in a specific field:  `status:200`
 - Find all from 400-499 status codes: `status:[400 TO 499]`
 - Find status codes 400-499 with the extension php: `status:[400 TO 499] AND extension:PHP`
 - You can read more about the Lucene Query String syntax in the [Lucene documentation](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html)

  
