# Wireless-ElasticStack-Ingest
This repository presents a method to ingest PCAP files that were collected via Airdump-ng and Kismet

We Start off by converting a PCAP with 802.11 wireless packets to JSON
<pre><code>sudo tshark -T ek -r "pcap_file" > "wifi.json" </code></pre>

If we have a directory of 802.11 wireless PCAPs, we can loop through the PCAPs and create a JSON file for each one
<pre><code>ls -la *.cap | awk '{print $9}' | while read line ; do sudo tshark -T ek -r $line >> wifi_$line.json; done</code></pre>

To ensure that we can ingest the JSON file for Elastic Stack versions 7+, we need to strip out the line containing illegal fields in the JSON file
<pre><code>sed -i '/"index"/d' "wifi.json"</code></pre>

If we have a directory of JSON files, we  can loop through the files and delete the line containing the illegal fields in the JSON file
<pre><code>ls -la *.json | awk '{print $9}' | while read line ; do sed -i '/"index"/d' $line ; done</code></pre>

Now we are ready to ingest the JSON files using Logstash.  Please see the Logstash configuration file at: 
![Logstash Configuration](https://github.com/threathunternotebook/Wireless-ElasticStack-Ingest/blob/main/logstash_wlan.conf)

If you are using PCAP files captured with Kismet and GPS data, you need to to send a geolocation template to Elasticsearch.  You can find a template at ![wifi Geolocation Template](https://github.com/threathunternotebook/Wireless-ElasticStack-Ingest/blob/main/wifi-template.json)

You can send the template to Elasticsearch using the example below
<pre><code>curl -H 'Content-Type: application/json' -XPUT http://localhost:9200/_index_template/wifi-pcap -d @wifi-template.json</code></pre>


Enjoy!
