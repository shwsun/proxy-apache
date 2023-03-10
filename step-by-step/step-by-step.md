# step by step manual  
## 0. Overview  
System architecture  
![System architecture](../docs/images/rproxy-outline.png)   

## 1. web server A  
```bash
echo "<html><head><title> A </title><meta http-equiv='Content-Type' content='text/html; charset=utf-8' /></head><body><h1> A </h1></body></html>" > ~/index.html 
# nginx web server run with index.html  
sudo docker run -d --rm -v ~/index.html:/usr/share/nginx/html/index.html -p 8881:80 --name web-a nginx  
```

## 2. jupyter lab web server  
```bash
sudo docker run -itd --rm --name jupyter1 -p 8888:8888 shwsun/jupyter-spark jupyter lab --allow-root --ip='*' --NotebookApp.allow_origin='*' --NotebookApp.notebook_dir='/tf' --workspace='/tf' --NotebookApp.base_url='/jupyter1' --NotebookApp.trust_xheaders=True 
--NotebookApp.ip='localhost'
# to get login token  
sudo docker exec -it jupyter1 /bin/bash -c "echo 'this is #1.'>/tf/jupyter1.txt"
sudo docker exec -it jupyter1 jupyter server list  
```
## 3. jupyter lab web server 2    
```bash
sudo docker run -itd --rm --name jupyter2 -p 8889:8888 shwsun/jupyter-spark jupyter lab --allow-root --ip='*' --NotebookApp.allow_origin='*' --NotebookApp.notebook_dir='/tf' --workspace='/tf' --NotebookApp.base_url='/jupyter2' --NotebookApp.trust_xheaders=True 
# to get login token  
sudo docker exec -it jupyter2 /bin/bash -c "echo 'this is #2.'>/tf/jupyter2.txt"
sudo docker exec -it jupyter2 jupyter server list  
```
## 4. http proxy  
- run apache reverse proxy server  
```bash
# 3. apache docker run
sudo docker run -d --rm --name proxy -p 80:80 httpd
```

## 웹 사이트 확인  
1~4 까지를 실행하면, 아래와 같은 웹페이지를 확인할 수 있습니다.  
아직까지는 `proxy` container가 `/A`, `/jupyter1`, `/jupyter2`에 대한 reverse-proxy 역할을 하고 있지는 않는 상태 입니다.  
- <ip>:8881 
- <ip>:8888/jupyter  
- <ip>:8889/jupyter2  
- <ip>:80  
![Web pages](../docs/images/webpages.png)  

---  
## proxy setting  
`proxy` container가 나머지 3개의 서비스에 대한 외부 요청을 중재하는 단일 창구로 작동하게 reverse proxy 설정을 합니다.  
<ip>:80 에서 서비스하고 있는 Apache2 proxy 서버를 제외하고 나머지 서버는 내부에서만 접근 가능하고, 
외부에서는 `<ip>/A`, `<ip>/jupyter`, `<ip>` 로 접근 가능하게 프락시를 설정합니다.  

- 먼저, 위에서 생성한 4개 컨테이너의 내부 ip를 확인합니다. 
```bash
# network 목록 확인  
# sudo docker network ls 
# 기본으로 bridge 에 할당된다.  
sudo docker inspect bridge  
```
- 컨테이너 각각에 부여된 ip는 아래와 같습니다. 
![Container IPs](../docs/images/docker-network-ip.png)   


- `httpd-vhosts.conf` 파일 상단의 변수 선언부를 확인한 ip에 맞게 수정합니다.  
- 수정한 설정 파일을 apache 서버에 배포합니다.  
```bash
# 미리 준비해 둔 설정파일 proxy 서버 설정파일 경로에 복사  
sudo docker cp httpd.conf proxy:/usr/local/apache2/conf/httpd.conf
sudo docker cp httpd-vhosts.conf proxy:/usr/local/apache2/conf/extra/httpd-vhosts.conf  
# apache restart 
sudo docker exec -it proxy apachectl restart

```

- <ip>  : `httpd-vhosts.conf` 에서 `jupyter1`으로 redirect 해서, `jupyter1` 요청으로 처리 됩니다.  
- <ip>/A : `A` 웹 페이지가 서비스 됩니다.  
- <ip>/jupyter1 : `jupyter1` 노트북이 서비스 됩니다. python notebook 과 terminal 모두 정상 작동해야 합니다.  
- <ip>/jupyter2 : `jupyter2` 노트북이 서비스 됩니다. python notebook 과 terminal 모두 정상 작동해야 합니다.  


