    // 1. 快照是增量的
    // 2. 快照无法保证数据一致性
    // 3. 快照可以用来快速部署相同的实例，比parallelssh方便很多
    // 4. A是基备，B是A上的增备，C是B上的增备，则删除B不会影响C的使用，即依然可以把硬盘恢复到C的状态
        
        
#### 一. 使用API创建快照 ####

```python

#!/usr/bin/python
# -*- coding:utf-8 -*-

""" This script used to create a snapshot for CBS in qcloud """

import time
import random
import base64
import hmac
import hashlib
import urllib2
import urllib
import json

def create_paras(region, secretid, diskid, snapname):
	paras = {
		'Action' 		: 	'CreateSnapshot',
		'SecretId' 		: 	secretid,
		'Timestamp' 		: 	int(time.time()),
		'Nonce' 		: 	random.randint(10000, 99999),
		'Region' 		: 	region,
		'SignatureMethod' 	: 	'HmacSHA256',
		'storageId' 		: 	diskid,
		'snapshotName' 	: 	snapname
	}
	return paras

def create_signature(paradict, host, path, secretkey):
	paralist = sorted(paradict.iteritems(), key = lambda kv : kv[0], reverse = False)
	n = len(paralist)
	i = 0
	while i < n:
		paralist[i] = "%s=%s" % (paralist[i][0], paralist[i][1])
		i += 1
	srcstr = 'GET' + host + path + '&'.join(paralist)
	return base64.b64encode(hmac.new(secretkey, srcstr, digestmod = hashlib.sha256).digest())

def create_url(host, path, paradict, signature):
	paradict['Signature'] = signature
	paralist = []
	for k, v in paradict.iteritems():
		paralist.append("%s=%s" % (k, urllib.quote(str(v).encode('utf-8', 'replace'))))
	return "https://%s%s%s" % (host, path, '&'.join(paralist))

def action(secretid, secretkey, host, path, region, diskid, snapname):
	# 1. create parameters
	paras = create_paras(region, secretid, diskid, snapname)

	# 2. create signature
	signature = create_signature(paras, host, path, secretkey)

	# 3. create url
	url = create_url(host, path, paras, signature)
	
	# 4. request
	req = urllib2.Request(url)
	try:
		res = json.loads(urllib2.urlopen(req).read())
		print res
	except Exception, e:
		print "!-- failed to create a snapshot"
		print str(e)

if __name__ == '__main__':
	# define variables ( not often change )
	my_secretid = 'xxx'
	my_secretkey = 'yyy'
	my_host = 'snapshot.api.qcloud.com'
	my_path = '/v2/index.php?'

	# define variables ( often change )
	my_region = 'sh'
	my_diskid = 'disk-296vtjv3'
	my_snapname = 'snap01'

	action(my_secretid, my_secretkey, my_host, my_path, my_region, my_diskid, my_snapname)

```

#### 二. 快照回滚 ####

        1. 建议直接使用mount和umount，而不是改/etc/fstab
        2. 回滚之前，要umount
        3. 回滚之前，要在cvm上卸载该cbs
