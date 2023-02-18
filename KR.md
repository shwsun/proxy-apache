# Apache2 리버스 프락시 설정하기  
apache http reverse proxy setting(apache + 정적 웹 페이지 + 동적 웹 사이트) 
Apache2를 이용해서 동적 http 서버 서비스하기  
서버는 아래와 같이 구성  
- Apache2(Reverse proxy server)  
- Web A : static web service  
- Jupyter : Jupyter lab web UI  

- http://<your ip> - apache index html page  
- http://<your ip>/A - index.html in <second ip>:<second port>  
- http://<your ip>/jupyter - jupyter lab web page in <jupyter ip>:<jupyter port>  


## step by step  
apache http 서버를 docker 로 실행하고, A web server와 Jupyter를 각각 도커로 실행하는 과정을 명령창을 통해 직접 설치하는 방식  

### 1. Apache2 컨테이너 실행하기  

```bash
# 3. apache docker 실행  
sudo docker run -d --rm --name proxy -p 80:80 httpd
# sudo docker run -d --rm --name proxy -p 80:80  httpd:alpine3.17
sudo docker exec -it proxy /bin/bash

# 필수 라이브러리 설치 : vi, ps  
apt-get update 
apt-get install -y vim 
apt-get install -y procps
# hosts 편집  

# httpd conf 편집  : ServerName, mod_proxy, mod_proxy_http, access_log, error_log , Include vhosts.conf
vi /usr/local/apache2/conf/httpd.conf  
# mod_proxy.so, mod_proxy_http.so, mod_proxy_wstunnel.so
vi /usr/local/apache2/conf/extra/httpd-vhosts.conf  

# apache 재시작  
/usr/local/apache2/bin/apachectl restart
```




