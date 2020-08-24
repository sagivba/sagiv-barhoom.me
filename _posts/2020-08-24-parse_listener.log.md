---
layout: post
title:  "parsing  Oracle listener.log"
author: "Sagiv Barhoom"
date:   2020-08-24
categories: Python
background: '/img/posts/python.jpg'
---
# Parsing  Oracle ```listener.log``` file 

My goal here was to map from the listener.log file the hostnames, programs and OS users that connect to the DB.
where is the listener.log located?
## Setting the data
``` bash 
locate listener.log
```
Let's get only  relevant lines from ``` listener.log ``` - that's means lines:
* With the string  'CONNECT' 
* With the string  ```$SID``` 
* From the year 2020 string 

```bash 
$ echo $SIDmyDB
$ grep CONNECT | grep -i "=$SID" | grep -P '\w\w\w-2020'> listener-clean.log
```

##  Aggregation into json
Now lets aggregate the data from ```listener-clean.log``` into JSON files.

```python
import json
import os
import re
import socket
from builtins import tuple
from pprint import pprint

def parse_connect_data_record(line, date):
    keyword_re_template = "{}=[^)]+[)]"
    program_re = keyword_re_template.format('PROGRAM')
    host_re = keyword_re_template.format('HOST')
    user_re = keyword_re_template.format('USER')
    programs = tuple(map(lambda s: s.replace('PROGRAM=', '').replace(')', '', ).lower(), re.findall(program_re, line)))
    programs =  tuple(map(lambda s:os.path.basename(s),programs))
    hosts = tuple(map(lambda s: s.replace('HOST=', '').replace(')', '', ).lower(), re.findall(host_re, line)))
    users = tuple(map(lambda s: s.replace('USER=', '').replace(')', '', ).lower(), re.findall(user_re, line)))

    for u in users:
        if len(programs)==0:
            # print ("WARNNIG: programs {} is more then one! line=>>>{}<<<".format(programs,line))
            programs=None
        elif len(programs)>1:
            print ("WARNNIG: programs {} is more then one!".format(programs))
        else:
            programs=programs[0]
        key_p,key_u,key_h=str(programs),str(u),str(hosts)
        data['PROGRAMS'].setdefault(key_p,{})
        data['PROGRAMS'][key_p].setdefault(key_h,{})
        data['PROGRAMS'][key_p][key_h].setdefault(key_u,0)
        data['PROGRAMS'][key_p][key_h][key_u]+=1

        data['HOSTS'].setdefault(key_h,{})
        data['HOSTS'][key_h].setdefault(key_p,0)
        data['HOSTS'][key_h][key_p]+=1

def parse_line(line, data):
    if 'CONNECT_DATA' in line:
        parse_connect_data_record(line, date)
    else:
        print("NOT SUPPORTED: {}".format(line))


def parse_log(data,filepath):
    with open(filepath) as fp:
        line = fp.readline()
        cnt = 1
        while line:
            # print("Line {}: {}".format(cnt, line.strip()))
            parse_connect_data_record(line, data)
            line = fp.readline()
            cnt += 1
            if cnt % 100000 ==0 :
                print ("{}%".format(int(100*cnt/19077352)))

def create_output_file(data,programs_filepath,hosts_filepath):
    with open(programs_filepath, 'w') as fp:
        json.dump(data["PROGRAMS"],fp, indent=4, sort_keys=True)

    with open(hosts_filepath, 'w') as fh:
        json.dump(data["HOSTS"],fh, indent=4, sort_keys=True)

if __name__ == "__main__":
    data = {'PROGRAMS' : {}, 'HOSTS' : {}}


    filepath = r'/tmp/listener-2020.log'
    programs_filepath=r'/tmp/PROGRAMS-2020.json'
    hosts_filepath=r'/tmp/HOSTS-2020.json'
    parse_log(data,filepath)
    create_output_file(data,programs_filepath,hosts_filepath)
```

At this stage we have  two files: 
1. Thr file ```HOSTS-2020.json``` which contains records that look like :

```json
"('srv1.domain.com', '147.222.20.33')": {
    "myprogram.import.exe": 72,
    "ourprogram.import.exe": 1463
},
```
2. The file ```PROGRAMS-2020.json``` which contains records that look like :

```json
"ourprogram.import.exe": {
    "('srv1.domain.com', '147.222.20.33')": {
        "some_username": 1463
    }
 },
 ```
   
 ## Replacing IP addresses with hostnames
 ```python
import re
import socket

def ip_to_host(ip):
    try:
        hostname = socket.gethostbyaddr(ip)[0]
        line = line.replace(ip, hostname)
    except Exception as e:
        print("ERROR:ip={}: {}".format(ip, e))

def get_hostnames_from_ips(filepath):
    out_lines = []
    with open(filepath) as fp:
        line = fp.readline()
        cnt = 1
        while line:
            # print("Line {}: {}".format(cnt, line.strip()))
            ips = re.findall(r'[0-9]+(?:\.[0-9]+){3}', line)
            for ip in ips:
                ip_to_host(ip)
            out_lines.append(line)

            line = fp.readline()
            cnt += 1
    return out_lines

def create_file(filepath,lines):

    with open(filepath, 'w') as fh:
        fh.write("".join(lines))

if __name__ == "__main__":
    hosts_filepath = '/tmp/HOSTS-2020.json'
    programs_filepath = r'/tmp/PROGRAMS-2020.json'

    lines=get_hostnames_from_ips(hosts_filepath)
    create_file(hosts_filepath.replace(".json","-hosts.json"),lines)

    lines = get_hostnames_from_ips(programs_filepath)
    create_file(programs_filepath.replace(".json","-hosts.json"),lines)
    
 ```

The above code will create two files in which IP addresses will be replaced by hostname when possible: 
1. The file ```HOSTS-2020-hosts.json``` which contains records that look like:

```json
"('srv1.domain.com', 'srv1.domain.com')": {
    "myprogram.import.exe": 72,
    "ourprogram.import.exe": 1463
},
```
2. The file ```PROGRAMS-2020-hosts.json``` which contains records that look like:
```json
"ourprogram.import.exe": {
    "('srv1.domain.com', 'srv1.domain.com')": {
    "some_username": 1463
    }
},
```
