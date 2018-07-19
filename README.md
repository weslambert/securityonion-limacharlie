# securityonion-limacharlie
Send logs from LimaCharlie to Security Onion   
*PLEASE test with a test instance of Security Onion before attempting in Production*

1. Sign up for a free account at https://app.limacharlie.io.
2. Download the sensor package and follow the instructions to activate the sensor on a supported OS.
3. Create your detection rules.
3. Add an output for `s3` or `syslog`, selecting to alert on Detections.  If using `syslog` make sure to send using TLS and specify your IP, then a port of 6514, like: `SECURITYONIONIP:6514`.    
4. Make sure to allow LimaCharlie logs via Security Onion (Example, for syslog: `sudo ufw allow proto tcp from LIMACHARLIE_IP to any port 6514`) and your firewall (make sure to configure your Security Group if AWS).    
5. On a test standalone instace, copy the files in the `s3` or `syslog` folder to `/etc/logstash/custom`, with the exception of securityonion.conf (this file is only included as a reference for how you should expose logstash-limacharlie-template.json through `LOGSTASH_OPTIONS`).
6. If using the `s3` input, specify the bucket name, access key id, and secret access key in `0000_limacharlie_s3.conf`.

    If using the `syslog` input, you'll need to add configuration to `/etc/syslog-ng/syslog-ng.conf` and create a cert to send syslog    securely:


    ````
    source s_tls {
                tcp(port(6514)
                tls( key_file("/etc/syslog-ng/ssl/logserver.key")
                        cert_file("/etc/syslog-ng/ssl/logserver.crt")
                peer_verify(optional-untrusted))
                program_override("limacharlie")
                flags(no-parse)
                );
    };
    ````    


   Also add `source(s_tls);` to the `log {}` stanza with the other sources like bro, ossec, network, etc.    
   
   The following link is what I've used to create the cert:
   
   https://www.logzilla.net/2014/10/17/configuring-tls-tunnels-in-syslog-ng.html
   
   Restart syslog-ng with `sudo service syslog-ng restart`.    

6. Run the following commands to remove old templates, and to sync the files and restart Logstash:   
`sudo so-logstash-stop`   
`curl -XDELETE localhost:9200/_template/logstash*`   
`sudo so-logstash-restart && sudo tail -f /var/log/logstash/logstash.log`
7. Wait until Logstash log shows "Pipelines running...".   
8. Trigger a detection rule on the host with sensor installed and wait for the event to flow through Logstash/Kibana (try filtering events with `event_type:limacharlie`.
