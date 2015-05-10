+++
date = "2015-05-10T23:44:43+08:00"
menu = "main"
tags = ["subjectAltName"]
title = "DNSPOD API SSL证书调整(subjectAltName)引起的报错"

+++

报错信息如下：
	
	Python 2.7 (r27:82500, Jan  7 2014, 23:14:35)
	[GCC 4.1.2 20080704 (Red Hat 4.1.2-50)] on linux2
	Type "help", "copyright", "credits" or "license" for more information.
	>>> import requests
	>>> r = requests.get("https://dnsapi.cn")
	Traceback (most recent call last):
  		File "<stdin>", line 1, in <module>
  		File "/usr/local/lib/python2.7/site-packages/requests-2.0.0-py2.7.egg/requests/api.py", line 55, in get
    		return request('get', url, **kwargs)
  		File "/usr/local/lib/python2.7/site-packages/requests-2.0.0-py2.7.egg/requests/api.py", line 44, in request
    		return session.request(method=method, url=url, **kwargs)
  		File "/usr/local/lib/python2.7/site-packages/requests-2.0.0-py2.7.egg/requests/sessions.py", line 361, in request
    		resp = self.send(prep, **send_kwargs)
  		File "/usr/local/lib/python2.7/site-packages/requests-2.0.0-py2.7.egg/requests/sessions.py", line 464, in send
    		r = adapter.send(request, **kwargs)
  		File "/usr/local/lib/python2.7/site-packages/requests-2.0.0-py2.7.egg/requests/adapters.py", line 363, in send
    		raise SSLError(e)
	requests.exceptions.SSLError: hostname 'dnsapi.cn' doesn't match u'www.dnspod.cn'

DNSPOD当前证书截图：
![dnspod](http://m114-static.qiniudn.com/img/dnspod_cert.png)

证书信息查看

	openssl s_client -connect dnsapi.cn:443 | tee dnsapi.cert
	openssl x509 -inform PEM -in dnsapi.cert -text

Python SSL验证

	Python 2.7 (r27:82500, Jan  7 2014, 23:14:35)
	[GCC 4.1.2 20080704 (Red Hat 4.1.2-50)] on linux2
	Type "help", "copyright", "credits" or "license" for more information.
	>>>  import _ssl
	>>> _ssl._test_decode_cert('dnsapi.cert')
	{'notBefore': 'Mar 24 00:00:00 2015 GMT', 'serialNumber': '3E4F6D495C407163586D0B900A007555', 'notAfter': 'May 12 23:59:59 2015 GMT', 'version': 3, 'subject': ((('1.3.6.1.4.1.311.60.2.1.3', u'CN'),), (('1.3.6.1.4.1.311.60.2.1.2', u'ShanDong'),), (('1.3.6.1.4.1.311.60.2.1.1', u'YanTai'),), (('2.5.4.15', u'Private Organization'),), (('serialNumber', u'370635200013814'),), (('countryName', u'CN'),), (('stateOrProvinceName', u'\u5c71\u4e1c\u7701'),), (('localityName', u'\u70df\u53f0\u5e02'),), (('organizationName', u'DNSPod, Inc.'),), (('organizationalUnitName', u'IT'),), (('commonName', u'www.dnspod.cn'),)), 'issuer': ((('countryName', u'US'),), (('organizationName', u'GeoTrust Inc.'),), (('commonName', u'GeoTrust Extended Validation SSL CA - G2'),))}


	Python 2.7.6 (default, Sep  9 2014, 15:04:36)
	[GCC 4.2.1 Compatible Apple LLVM 6.0 (clang-600.0.39)] on darwin
	Type "help", "copyright", "credits" or "license" for more information.
	>>> import _ssl
	>>> _ssl._test_decode_cert('dnsapi.cert')
	{'subjectAltName': (('DNS', 'www.dnsapi.cn'), ('DNS', 'monitor.dnspod.cn'), ('DNS', 'api.dnspod.com'), ('DNS', 'tickets.dnspod.com'), ('DNS', 'ec.dnspod.cn'), ('DNS', 'domains.dnspod.cn'), ('DNS', 'support.dnspod.cn'), ('DNS', 'm.dnspod.cn'), ('DNS', 'ssl.ptlogin2.dnspod.cn'), ('DNS', 'monitor.dnspod.com'), ('DNS', 'support.dnspod.com'), ('DNS', 'tickets.dnspod.cn'), ('DNS', 'statics.dnspod.cn'), ('DNS', 'stat.dnspod.cn'), ('DNS', 'www.dnspod.com'), ('DNS', 'dnspod.com'), ('DNS', 'static.dnspod.com'), ('DNS', 'libs.dnspod.cn'), ('DNS', 'blog.dnspod.cn'), ('DNS', 'dnsapi.cn'), ('DNS', 'www.dnspod.cn'), ('DNS', 'dnspod.cn')), 'notBefore': 'Mar 24 00:00:00 2015 GMT', 'serialNumber': '3E4F6D495C407163586D0B900A007555', 'notAfter': 'May 12 23:59:59 2015 GMT', 'version': 3, 'subject': ((('1.3.6.1.4.1.311.60.2.1.3', u'CN'),), (('1.3.6.1.4.1.311.60.2.1.2', u'ShanDong'),), (('1.3.6.1.4.1.311.60.2.1.1', u'YanTai'),), (('businessCategory', u'Private Organization'),), (('serialNumber', u'370635200013814'),), (('countryName', u'CN'),), (('stateOrProvinceName', u'\u5c71\u4e1c\u7701'),), (('localityName', u'\u70df\u53f0\u5e02'),), (('organizationName', u'DNSPod, Inc.'),), (('organizationalUnitName', u'IT'),), (('commonName', u'www.dnspod.cn'),)), 'issuer': ((('countryName', u'US'),), (('organizationName', u'GeoTrust Inc.'),), (('commonName', u'GeoTrust Extended Validation SSL CA - G2'),))}

解决方法：

**`升级Python至2.7.3以上版本,低版本不支持ssl的subjectAltName特性`**

来源参考: https://github.com/shazow/urllib3/issues/523