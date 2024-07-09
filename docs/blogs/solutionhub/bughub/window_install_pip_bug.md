---
title: window环境中pip安装报错
date: 2022-01-05 23:51:41
tags: 
  - Bug
---

- 错误日志
```bash
(blogenv) λ pip install mkdocs -i https://pypi.mirrors.ustc.edu.cn/simple/ --trusted-host pypi.douban.com       
Looking in indexes: https://pypi.mirrors.ustc.edu.cn/simple/                                                    
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken 
by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1131)'))': /simple/mkdocs/           
WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken 
by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1131)'))': /simple/mkdocs/           
WARNING: Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken 
by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1131)'))': /simple/mkdocs/           
WARNING: Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken 
by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1131)'))': /simple/mkdocs/           
WARNING: Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken 
by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1131)'))': /simple/mkdocs/           
Could not fetch URL https://pypi.mirrors.ustc.edu.cn/simple/mkdocs/: There was a problem confirming the ssl cert
ificate: HTTPSConnectionPool(host='pypi.mirrors.ustc.edu.cn', port=443): Max retries exceeded with url: /simple/
mkdocs/ (Caused by SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1131)'))) - skipping  
ERROR: Could not find a version that satisfies the requirement mkdocs                                           
ERROR: No matching distribution found for mkdocs                                                                
Could not fetch URL https://pypi.mirrors.ustc.edu.cn/simple/pip/: There was a problem confirming the ssl certifi
cate: HTTPSConnectionPool(host='pypi.mirrors.ustc.edu.cn', port=443): Max retries exceeded with url: /simple/pip
/ (Caused by SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1131)'))) - skipping        
```

- 解决方式
```bash
pip install mkdocs -i http://pypi.douban.com/simple --trusted-host pypi.douban.com
```
