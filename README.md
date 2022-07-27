# node-express-cloud-native

### 1. Simple Express app

Use `npx express-generator` to build a simple express app

Run `npm start` to bring up the app on http://localhost:3000


### 2. Docker setup and running

Install docker for desktop.

Use "wget" to download individual files using:<br>
```sh
wget https://raw.githubusercontent.com/NodeShift/docker/main/{file}
```

Use the following command to build the docker image:<br>
```sh
docker build -t nodeserver -f Dockerfile .
```

To run as an interactive application on your command line:<br>
```sh
docker run -i -p 3000:3000 -t nodeserver`
```
This maps port 3000 in the Docker image to port 3000 on your machine. If you are using a different port, you will need to change the mapping.

To run as a daemon process:<br>
```sh
docker run -d -p 3000:3000 -t nodeserver`
```
This uses the -d flag rather than the -i flag to run the Docker image as a background task.

To push the image to docker hub<br>
```sh
docker login
```

Tag the docker image<br>
```sh
docker tag nodeserver-run <namespace>/nodeserver:1.0.0
```

Publish your application's Docker image to DockerHub:<br>
```sh
docker push <namespace>/nodeserver:1.0.0
```

### 3. Setup Kubernetes

Enable Kubernetes from Docker for desktop<br>

Make sure you allocate atleast 6 CPUs and 8 GB memory

###4. Add Helm

Use brew formula to install helm latest version<br>

Copy charts manually from https://github.com/nodeshift/helm

In order to change the image.repository parameter, open the charts/nodeserver/values.yaml file and change the following entry:<br>
```sh
image: 
    repository: <namespace>/nodeserver
```

to set <namespace> to your namespace on DockerHub where you published your application as nodeserver.


Run the following to install nodeserver app:<br>
```sh
helm install nodeserver chart/nodeserver
```

Find the deployment using `helm list --all` and searching for an entry with the chart name "nodeserver".<br>
Remove the application with `helm uninstall nodeserver`<br>

Deploy multiple instances by changing the replicaCount in the values.yml


##5. Add liveness and readiness checks

Register /live and /ready endpoints to check for the liveness and readiness using pingcheck as below:<br>

```sh
let pingcheck = new health.PingCheck("example.com");
healthcheck.registerReadinessCheck(pingcheck);

app.use('/live', health.LivenessEndpoint(healthcheck))
app.use('/ready', health.ReadinessEndpoint(healthcheck))
```


##6. Support for metrics by Prometheus and Grafana

Use helm chart to install prometheus and grafana from prometheus-community

Set hostRootFsMount.enabled=false for node exporter:
```sh
helm install prome prometheus-community/kube-prometheus-stack --set prometheus-node-exporter.hostRootFsMount.enabled=false
```

Forward the prometheus and grafana ports to see the UI from localhost:
```sh
kubectl port-forward svc/prome-kube-prometheus-stac-prometheus 9090
kubectl port-forward svc/prome-grafana 3000:80
```

Grafana credentials can be obtained by the following command:
```sh
kubectl get secret prome-grafana -o yaml
admin-password: cHJvbS1vcGVyYXRvcg==  | prom-operator 
admin-user: YWRtaW4=    | admin
```


##7.Support for request tracking using zipkin and jaeger

Use the following to download jaeger-all-in one charts:<br>
```sh
helm repo add jaeger-all-in-one https://raw.githubusercontent.com/hansehe/jaeger-all-in-one/master/helm/charts
helm install jaeger-all-in-one jaeger-all-in-one/jaeger-all-in-one
```






