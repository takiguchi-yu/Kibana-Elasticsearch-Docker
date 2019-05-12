# dstat

## dstat 起動スクリプト作成
### run-dstat.sh
```
#!/bin/sh

dstat -T --cpu --disk -D xvda1 --load --mem --net --swap -g --output /home/ec2-user/dstat/dstat-`date +%Y%m%d-%H%M%S`.csv 1 3599
```
### crontab 設定
```
crontab -e
0 * * * * /home/ec2-user/dstat/run-dstat.sh > /dev/null
```
