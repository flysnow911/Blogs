### ns创建文件
namespace_create.yaml	
	
	apiVersion: v1
	kind: Namespace
	metadata:
	   name: development
	   labels:
		 name: development

### Pod部署文件
deployment.yaml

	apiVersion: apps/v1
	kind: Deployment
	metadata:
	   name: hello-world
	   labels: 
		 name: hello-world
	   namespace: development
	spec:
	  containers:
	  - env:
		- name: APOLLO_META
		  value: http://10.66.72.189:8080,http://10.66.72.190:8080
		- name: appId
		  value: hello-world
		image: registry/hello-world:0.0.1
		imagePullPolicy: Always
		name: hello-world
		ports:
		- containerPort: 80
		  hostPort: 80 
		  protocol: TCP
	  dnsPolicy: ClusterFirst

### Service创建文件service.yaml
	
	apiVersion: v1
	kind: Service
	metadata:
	  name: my-service
	  namespace: development
	spec:
	  selector:
		app: my-service
	  ports:
		- name: http
		  protocol: TCP
		  port: 80
		  targetPort: 8080
      selector:
		app: hello-world
	  type: ClusterIP

### Ingress文件ingress.yaml
	apiVersion: extensions/v1beta1
	kind: Ingress
	metadata:
	  name: basic-ingress
	  namespace: development
	spec:
	  rules:
	  - host: hello-world.dev.com //这个需要在客户端配置host,ip是nginx-controller的ip
	    http:
		  paths: 
		  - backend:
			  serviceName: hello-world
			  servicePort: 80
			path: /
		  
