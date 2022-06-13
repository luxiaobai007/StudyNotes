# Android

```yml
kind: pipeline
branches: [master, rc]


trigger:
  branch:
    - master
    - rc

steps:
   #1.gradle打包
	- name: gradle-build-rc
	pull: if-not-exists
    image: androidsdk/android-30
    volumes:
        - name: cache
        path: /root/.gradle
         #挂载宿主机的目录
    - name: data
    path: /home
    commands:
		- pwd
		- java -version
		- chmod +x ./gradlew
		- chmod +x ./upload
		- echo "sdk.dir=/opt/android-sdk-linux" > local.properties # 指定sdk的位置
		- echo "debug.API_SERVER=http://192.168.3.139:10003" >> local.properties
		- ./gradlew assembleDebug
		- ls -al app/build/outputs/apk/debug
		- ./upload ./app/build/outputs/apk/debug http://192.168.3.139:10003
	when:
    branch:
        - rc
 # 3. 钉钉通知
    - name: dingtalk
    image: guoxudongdocker/drone-dingtalk
	settings:
		token:
		from_secret: token
		message_color: true
		type: markdown
		message_pic: true
		sha_link: true
		when:
		status:
			- success
			- changed
			- failure

   # 挂载的主机卷，可以映射到docker容器中
   volumes:
	 # maven构建缓存
	   - name: cache
		   host:
			path: /home/ubuntu/project/antl/antl-android
			- name: data
			host:
			path: /home/ubuntu/project/antl/antl   #这边需要改成drone服务器上项目对应的文件路径
			- name: hosts
			host:
			path: /etc/hosts
																										
```





# JAVA

```yml
kind: pipeline
branches: [master, rc, dev]

trigger:
  branch:
    - master
    - rc

steps:
  # 1.gradle打包
- name: gradle-build-rc
pull: if-not-exists
image: gradle:jdk8
volumes:
	- name: cache
	path: /root/.gradle
#挂载宿主机的目录
	- name: data
	path: /home
	 commands:
		- pwd
		- cp /home/antl-backend/application-dev.properties src/main/resources/application-prod.properties
		- gradle build -x test
		- mv `ls -1r build/libs/*.jar | head -n 1` /drone/src/build/libs/backend.jar
when:
	branch:
		- rc
#2.项目打包部署
- name: deploy-rc
	image: drillster/drone-rsync
	volumes:
		- name: hosts
		path: /etc/hosts
	- name: data
		path: /home
	settings:
		hosts:
			- antl-dev
		user:
			from_secret: ubuntu
		key:
			from_secret: ubuntu_key
		delete: true
		source: /drone/src/build/libs/.
		target: /home/ubuntu/backend
		include: [ "backend.jar"]  #只需要编译出来java 即可
		exclude: [ "*" ]
		script:
			- pwd
			- echo "正在停止旧线程"
			- sudo systemctl restart antl.service
			- echo "正在重启服务器.. 请等待启动完成"
when:
	branch:
		- rc

  #3. 钉钉通知
- name: dingtalk
image: guoxudongdocker/drone-dingtalk
settings:
	token:
	from_secret: token1
	message_color: true
	type: markdown
	message_pic: true
	sha_link: true
	when:
	status:
		- success
		- changed
		- failure

# 挂载的主机卷，可以映射到docker容器中
volumes:
 # maven构建缓存
	- name: cache
	host:
	path: /var/lib/cache
  - name: data
  host:
  path: /home/ubuntu/project/antl   #这边需要改成drone服务器上项目对应的文件路径
  - name: hosts
  host:
  path: /etc/hosts

```





# 前端

```yml
kind: pipeline
branches: [master, rc]
name: antl

trigger:
  branch:
    - rc
    - master

steps:
  - name: restore-cache
    image: drillster/drone-volume-cache
    volumes:
      - name: cache
        path: /cache
    settings:
      restore: true
      mount:
        - ./node_modules

   #编译打包并做映射
     - name: build
	image: node:lts-stretch-slim
	volumes:
		- name: localtime
		path: /etc/localtime
		commands:
			- npm -v
			- node -v
			- npm install
			- npm run build
	
# 项目部署
- name: deploy-rc
	image: drillster/drone-rsync
volumes:
	- name: hosts
path: /etc/hosts
settings:
	hosts:
		- 192.168.x.x
	user:
		from_secret: user_secret
	key:
		from_secret: user_secret_key
	delete: true
	source: ./dist
	target: /home/ubuntu/www/
when:
	branch:
		- rc
#钉钉通知
- name:	 dingtalk
image: guoxudongdocker/drone-dingtalk
settings:
	token:
		from_secret: dingtalk_token1
		message_color: true
	type: markdown
	message_pic: true
	sha_link: true
when:
   status:
	- success
	- changed
	- failure



- name: rebuild-cache
	image: drillster/drone-volume-cache
volumes:
	- name: cache
	path: /cache
settings:
	rebuild: true
mount:
	- ./node_modules


volumes:
	- name: cache
	host:
	path: /var/local/drone-cache
		- name: hosts
	host:
	path: /etc/hosts

```





# GO

```yml
kind: pipeline
branches: [master, rc]

trigger:
  branch:
    - master
    - rc

steps:
   #编译 拷贝二进制文件等相关文件到指定目录
     - name: go-build
	image: golang:1.15.8
	environment:
	CGO_ENABLED: 1
	GOPROXY: https://goproxy.io,direct
	volumes:
		- name: cache
	path: /go
		- name: data
		path: /home
	commands:
		- go build -ldflags "-s -w" -o ./build/antl-node
		- ls -al
# 将打包后的文件复制到宿主机映射目录

# 拷贝相关文件到目标主机
- name: deploy-rc
image: drillster/drone-rsync
volumes:
	- name: hosts
path: /etc/hosts
	- name: data
path: /home
settings:
	hosts:
		- 192.168.3.xxx
	ports:
		- 22
	user: ubuntu
	key:
		from_secret: deploy_key
	delete: false
	source: /drone/src/build/.                 #本机
	target: /home/ubuntu/antl-node             #远程主机
include: [ "*" ]
script:
	- sudo systemctl restart antl-node.service
	- sleep 1s
	- echo "正在重启服务器.. 请等待启动完成"
when:
	branch:
		- rc

#4. 钉钉通知
- name: dingtalk
	image: guoxudongdocker/drone-dingtalk
settings:
	token:
	from_secret: token
	message_color: true
	type: markdown
	message_pic: true
	sha_link: true
	when:
status:
	- success
	- changed
	- failure

# 挂载的主机卷，可以映射到docker容器中
volumes:
# maven构建缓存
	- name: cache
		host:
		path: /var/lib/cache
			- name: data
		host:
			path: /home/ubuntu/project/antl/antl-auto
		- name: hosts
		host:
		path: /etc/hosts

```



