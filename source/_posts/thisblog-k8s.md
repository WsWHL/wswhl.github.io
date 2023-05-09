---
title: Kubernetes This Blog
date: '2020-04-05 14:52:57'
updated: '2020-04-05 14:52:57'
categories:
  - Deploy
tags:
  - Blog
  - README
  - YAML
---
## 集群部署环境说明
	本文采用gcp云服务部署方案，1 个 vCPU，3.75 GB，单节点部署。

## 所需部署服务说明
- mysql, 有状态服务，用于数据持久化存储
- redis，主要用于文章数据缓存，以此提升网站响应速度
- nginx, 用于网站中静态文件发布服务器，与后端服务站点分离以此达到分流效果
- web, 后端站点发布服务
- ingress controller, 路由控制器，实现站点请求路由，以及站点tls配置
<!--more -->
## 部署

### #PersistentVolumeClaim
> 创建持久化存储卷，有效避免数据丢失。

- mysql pvc

	``` yaml
	apiVersion: v1
	kind: PersistentVolumeClaim
	metadata:
	  name: blog-volumeclaim
	spec:
	  accessModes:
		- ReadWriteOnce
	  resources:
		requests:
		  storage: 2Gi
	```
- static file pvc

	```yaml
	apiVersion: v1
	kind: PersistentVolumeClaim
	metadata:
	  name: blog-static-volumeclaim
	spec:
	  accessModes:
		- ReadWriteOnce
	  resources:
		requests:
		  storage: 100Mi
	```
- media file pvc

	``` yaml
	apiVersion: v1
	kind: PersistentVolumeClaim
	metadata:
	  name: blog-media-volumeclaim
	spec:
	  accessModes:
		- ReadWriteOnce
	  resources:
		requests:
		  storage: 512Mi
	```

### #ingress controller
1. 使用helm添加nginx ingress仓库地址
`helm repo add stable https://kubernetes-charts.storage.googleapis.com`
2. 编写nginx ingress controller配置

	``` yaml
	controller:
	  kind: Deployment
	  replicaCount: 1
	  updateStrategy: 
		rollingUpdate:
		  maxUnavailable: 1
		type: RollingUpdate
	  ingressClass: nginx
	  service:
		enabled: true
		enableHttp: true
		enableHttps: true
		type: LoadBalancer
	  resources:
		requests:
		  cpu: 20m
		  memory: 64Mi
		limits:
		  cpu: 50m
		  memory: 256Mi
	  defaultBackendService: default/web-service

	defaultBackend:
	  enabled: false

	rbac:
	  create: true
	```
	*** 这里的默认后台服务我是写的自己的站点，格式为namespace/service***
	
3. 安裝nginx ingress controller服务
`helm install nginx-ingress stable/nginx-ingress -f values.yaml`

### #mysql

- deployment

	``` yaml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: mysql
	spec:
	  selector:
		matchLabels:
		  app: mysql
	  strategy:
		type: Recreate
	  template:
		metadata:
		  labels:
			app: mysql
		spec:
		  containers:
		  - name: mysql
			image: mysql:latest
			args:
			- "--default-authentication-plugin=mysql_native_password"
			env:
			- name: MYSQL_DATABASE
			  value: "blog"
			- name: MYSQL_USER
			  value: "root"
			- name: MYSQL_ROOT_PASSWORD
			  value: "123456"
			ports:
			- containerPort: 3306
			  name: mysql
			volumeMounts:
			- name: mysql-persistent-storage
			  mountPath: /var/lib/mysql
			resources:
			  requests:
				cpu: 50m
				memory: 512Mi
		  volumes:
		  - name: mysql-persistent-storage
			persistentVolumeClaim:
			  claimName: mysql-volumeclaim
	```
- service

	``` yaml
	apiVersion: v1
	kind: Service
	metadata:
	  name: mysql-service
	spec:
	  type: ClusterIP
	  ports:
	  - port: 3306
	  selector:
		app: mysql
	```
	
### #redis

- deployment

	``` yaml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: redis-server
	spec:
	  selector:
		matchLabels:
		  app: redis-server
	  replicas: 1
	  template:
		metadata:
		  labels:
			app: redis-server
		spec:
		  containers:
		  - name: redis-server
			image: redis:latest
			ports:
			- containerPort: 6379
			resources:
			  requests:
				cpu: 10m
				memory: 100Mi
			  limits:
				cpu: 10m
				memory: 128Mi
	```
	
- service

	``` yaml
	apiVersion: v1
	kind: Service
	metadata:
	  name: redis-service
	spec:
	  type: NodePort
	  ports:
	  - port: 6379
	  selector:
		app: redis-serve
	```
	
### #nginx

- deployment

	``` yaml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: nginx-server
	spec:
	  selector:
		matchLabels:
		  app: nginx-server
	  replicas: 1
	  template:
		metadata:
		  labels:
			app: nginx-server
		spec:
		  volumes:
		  - name: nginx-volume
			configMap:
			  name: nginx-config
			  items:
			  - key: nginx.conf
				path: nginx.conf
		  - name: blog-static-storage
			persistentVolumeClaim:
			  claimName: blog-static-volumeclaim
		  - name: blog-media-storage
			persistentVolumeClaim:
			  claimName: blog-media-volumeclaim
		  containers:
		  - name: nginx-server
			image: nginx:latest
			ports:
			- containerPort: 80
			resources:
			  requests:
				cpu: 20m
				memory: 256Mi
			volumeMounts:
			- mountPath: /etc/nginx/conf.d
			  name: nginx-volume
			- name: blog-static-storage
			  mountPath: /usr/src/app/web/static
			- name: blog-media-storage
			  mountPath: /usr/src/app/web/media
	```
	** 需要创建`nginx.conf` configmap配置。*
	
- service

	``` yaml
	apiVersion: v1
	kind: Service
	metadata:
	  name: nginx-service
	  labels:
		app: nginx-service
	spec:
	  type: NodePort
	  ports:
	  - port: 80
		targetPort: 80
		protocol: TCP
		name: http
	  selector:
		app: nginx-server
	```
	
### #web

- deployment

	``` yaml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: web
	spec:
	  selector: 
		matchLabels:
		  app: web
	  replicas: 3
	  strategy:
		type: RollingUpdate
		rollingUpdate:
		  maxSurge: 25%
		  maxUnavailable: 25%
	  minReadySeconds: 3
	  template:
		metadata:
		  labels:
			app: web
		spec:
		  containers:
		  - name: web
			image: asia.gcr.io/whl-vps/blog_web:latest
			command: ["/bin/sh", "-c", "python manage.py rebuild_index --noinput; uwsgi -i uwsgi.ini"]
			env:
			- name: DEBUG
			  value: "0"
			- name: DOMAIN
			  value: "*"
			- name: EMAIL_USER
			  value: "9239****@qq.com"
			- name: EMAIL_PASSWORD
			  value: "*****"
			- name: EMAIL_PORT
			  value: "587"
			- name: MYSQL_HOST
			  value: "mysql-service"
			- name: MYSQL_PORT
			  value: "3306"
			- name: MYSQL_DATABASE
			  value: "blog"
			- name: REDIS_HOST
			  value: "redis-service"
			- name: REDIS_PORT
			  value: "6379"
			- name: REDIS_DB
			  value: "0"
			- name: SECRET_KEY
			  value: "*****"
			ports:
			- containerPort: 8000
			  protocol: TCP
			resources:
			  requests:
				cpu: 10m
				memory: 128Mi
			volumeMounts:
			- name: blog-static-storage
			  mountPath: /usr/src/app/web/static
			- name: blog-media-storage
			  mountPath: /usr/src/app/web/media
		  volumes:
		  - name: blog-static-storage
			persistentVolumeClaim:
			  claimName: blog-static-volumeclaim 
		  - name: blog-media-storage
			persistentVolumeClaim:
			  claimName: blog-media-volumeclaim
	```
	
- service

	``` yaml
	apiVersion: v1
	kind: Service
	metadata:
	  name: web-service
	spec:
	  type: NodePort
	  ports:
	  - port: 80
		targetPort: 8000
		protocol: TCP
	  selector:
		app: web
	```

- ingress

	``` yaml
	apiVersion: extensions/v1beta1
	kind: Ingress
	metadata:
	  name: web-ingress
	  annotations:
		kubernetes.io/ingress.class: nginx
		ingress.kubernetes.io/rewrite-target: /
		nginx.ingress.kubernetes.io/from-to-www-redirect: "true"
		nginx.ingress.kubernetes.io/ssl-passthrough: "true"
		nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
	spec:
	  tls:
	  - hosts:
		- www.thisblog.cn
		secretName: tls-secret
	  rules:
	  - host: www.thisblog.cn
		http:
		  paths:
		  - path: /
			backend:
			  serviceName: web-service
			  servicePort: 80
		  - backend:
			  serviceName: nginx-service
			  servicePort: 80
			path: /static
		  - backend:
			  serviceName: nginx-service
			  servicePort: 80
			path: /media
	```
	
- tls-secret

	``` yaml
	apiVersion: v1
	kind: Secret
	metadata:
	  name: tls-secret
	data:
	  tls.crt: <base64>
	  tls.key: <base64>
	```
	*** base64编码命令，`cat thisblog.cn.key | base64` or `cat thisblog.cn.crt | base64` ***
