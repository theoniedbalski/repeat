# Repeat [![CircleCI](https://circleci.com/gh/niedbalski/repeat.svg?style=svg)](https://circleci.com/gh/niedbalski/repeat)

*repeat* allows you to define a set of linux commands that needs to be run with a given periodicity and gather the output 
of those commands into a compressed tarball report for further analysis.
 
### Installation

Release artifacts can be found in the releases section [Github Releases](https://github.com/niedbalski/repeat/releases)
```shell script
wget -c https://github.com/niedbalski/repeat/releases/download/v0.0.1/repeat-0.0.1.linux-amd64.tar.gz -O - | tar -xz -C . --strip=1
./repeat --help
```
For installing the latest master build snap (edge channel):
```shell script
snap install --channel edge repeat --classic
```

For installing the latest stable version snap (stable channel):
```shell script
snap install --channel stable repeat --classic
```

Also docker images are available:

* Amd64 docker container [![Docker Repository on Quay](https://quay.io/repository/niedbalski/repeat-linux-amd64/status "Docker Repository on Quay")](https://quay.io/repository/niedbalski/repeat-linux-amd64)
* Arm64 docker container [![Docker Repository on Quay](https://quay.io/repository/niedbalski/repeat-linux-arm64/status "Docker Repository on Quay")](https://quay.io/repository/niedbalski/repeat-linux-arm64)

```shell script
docker run -v "$(pwd):/config" -it quay.io/niedbalski/repeat-linux-amd64:master --config /config/example_metrics.yaml
```

#### Command line

```shell script
usage: repeat --config=CONFIG [<flags>]

Flags:
  -h, --help             Show context-sensitive help (also try --help-long and --help-man).
  -l, --loglevel="info"  Log level: [debug, info, warn, error, fatal]
  -t, --timeout=0s       Timeout: overall timeout for all collectors
  -c, --config=CONFIG    Path to collectors configuration file
  -b, --basedir="/tmp"   Temporary base directory to create the resulting collection tarball
  -r, --results-dir="."  Directory to store the resulting collection tarball

```

#### Running with configuration

An example of running the collection for 5s (could be expressed in s,m,hours)

```shell script
repeat --config metrics.yaml --timeout=5s --results-dir=.
```

##### Example configuration

Note: Imports are allowed as http[s]/files, local collection names have precedence over imported ones.

```yaml
import:
  - https://raw.githubusercontent.com/niedbalski/repeat/master/example_metrics.yaml#md5sum=6c5b5d8fafd343d5cf452a7660ad9dd1
  - https://raw.githubusercontent.com/niedbalski/repeat/master/logs.yaml#md5sum=6c5b5d8fafd343d5cf452a7660ad9dd2
  - /tmp/other_collection.yaml

collections:
  process_list:
    command: ps auxh
    run-every: 2s
    exit-codes: any

  sockstat:
    command: cat /proc/sys/net/ipv4/tcp*mem /proc/net/sockstat
    run-every: 2s
    exit-codes: 0

  sar:
    run-once: true
    exit-codes: 0 127 126
    script: |
      #!/bin/bash

      echo "testing"

  uname:
    run-once: true
    script: |
      uname -a
```

This command will generate the following output:

```shell script

INFO[2020-04-23 17:57:13] Loading collectors from configuration file: example_metrics.yaml 
DEBU[2020-04-23 17:57:13] Importing item: https://raw.githubusercontent.com/niedbalski/repeat/master/example_metrics.yaml#md5sum=6c5b5d8fafd343d5cf452a7660ad9dd1 
INFO[2020-04-23 17:57:13] Scheduler timeout set to: 5.000000 seconds   
INFO[2020-04-23 17:57:13] Scheduling run of sleep collector every 2.000000 secs 
INFO[2020-04-23 17:57:13] Scheduling run of sockstat collector every 2.000000 secs 
INFO[2020-04-23 17:57:13] Scheduling run of sar collector every 0.000000 secs 
INFO[2020-04-23 17:57:13] Scheduling run of uname collector every 0.000000 secs 
INFO[2020-04-23 17:57:13] Scheduling run of lsof collector every 10.000000 secs 
INFO[2020-04-23 17:57:14] Running command for collector sleep          
INFO[2020-04-23 17:57:14] Running command for collector uname          
INFO[2020-04-23 17:57:14] Running command for collector lsof           
INFO[2020-04-23 17:57:14] Running command for collector sar            
INFO[2020-04-23 17:57:14] Running command for collector sockstat       
INFO[2020-04-23 17:57:14] Command for collector sleep, successfully ran, stored results into file: /tmp/repeat-044825910/sleep-2020-04-23-17:57:14 
INFO[2020-04-23 17:57:14] Command for collector uname, successfully ran, stored results into file: /tmp/repeat-044825910/uname-2020-04-23-17:57:14 
ERRO[2020-04-23 17:57:14] Command for collector lsof exited with exit code: exit status 255 - (not allowed by exit-codes config) 
INFO[2020-04-23 17:57:14] Command for collector sar, successfully ran, stored results into file: /tmp/repeat-044825910/sar-2020-04-23-17:57:14 
INFO[2020-04-23 17:57:14] Command for collector sockstat, successfully ran, stored results into file: /tmp/repeat-044825910/sockstat-2020-04-23-17:57:14 
INFO[2020-04-23 17:57:15] Running command for collector uname          
INFO[2020-04-23 17:57:15] Command for collector uname, successfully ran, stored results into file: /tmp/repeat-044825910/uname-2020-04-23-17:57:15 
INFO[2020-04-23 17:57:16] Running command for collector sockstat       
INFO[2020-04-23 17:57:16] Running command for collector uname          
INFO[2020-04-23 17:57:16] Running command for collector sleep          
INFO[2020-04-23 17:57:16] Command for collector sockstat, successfully ran, stored results into file: /tmp/repeat-044825910/sockstat-2020-04-23-17:57:16 
INFO[2020-04-23 17:57:16] Command for collector sleep, successfully ran, stored results into file: /tmp/repeat-044825910/sleep-2020-04-23-17:57:16 
INFO[2020-04-23 17:57:16] Command for collector uname, successfully ran, stored results into file: /tmp/repeat-044825910/uname-2020-04-23-17:57:16 
INFO[2020-04-23 17:57:17] Running command for collector uname          
INFO[2020-04-23 17:57:17] Command for collector uname, successfully ran, stored results into file: /tmp/repeat-044825910/uname-2020-04-23-17:57:17 
INFO[2020-04-23 17:57:18] Scheduler timeout (5.000000) reached, cleaning up and killing process 
INFO[2020-04-23 17:57:18] Cleaning up resources                        
INFO[2020-04-23 17:57:18] Creating report tarball at: repeat-report-2020-04-23-17-57.tar.gz  
Killed
```

### Contributing

Feel free to send PR(s) or reach niedbalski on #freenode or Telegram.