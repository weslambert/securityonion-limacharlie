# securityonion-limacharlie
Send logs from LimaCharlie to Security Onion   
*PLEASE test with a test instace of Security Onion before attempting in Production*

1. Sign up for a free account at https://app.limacharlie.io.
2. Download the sensor package and follow the instructions to activate the sensor on a supported OS.
3. Create your detection rules.
3. Add an output for s3 or syslog, selecting to alert on Detections.
4. Make sure to allow LimaCharlie logs via Security Onion (`sudo so-allow`) and your firewall (make sure to configure your Security Group if AWS).
5. On a test standalone instace, copy the files in the `s3` or `syslog` folder to `/etc/logstash/custom`, with the exception of securityonion.conf (this file is only included as a reference for how you should expose logstash-limacharlie-template.json through `LOGSTASH_OPTIONS`).
6. Run the following command to remove old templates, and to sync the files and restart Logstash:   
`sudo so-logstash-stop`   
`curl -XDELETE localhost:9200/_template/logstash*`   
`sudo so-logstash-restart && sudo tail -f /var/log/logstash/logstash.log`
7. Wait until Logstash log shows "Pipelines running...".   
8. Trigger a detecion rule on the host with sensor installed and wait for the event to flow through Logstash/Kibana (try filtering events with `event_type:limacharlie`.
