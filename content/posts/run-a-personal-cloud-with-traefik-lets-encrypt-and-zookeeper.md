---
title: "Run a personal Cloud with Traefik, Lets encrypt and Zookeeper"
date: 2019-08-16
draft: false
description: "Run a personal Cloud with Traefik, Lets encrypt and Zookeeper"
showToc: true
TocOpen: true
tags:
    - kubernetes
    - docker
    - cloud
    - gke
    - gitlab
    - google cloud
    - devops
    - personal-cloud
---
# Kubernetes ingress with Traefik
As mentioned in my [last blog post](/posts/kubernetes-for-sideprojects-hardware-is-dead) I want to focus on a provider neutral setup for my own cloud, using technology that is not bound to any cloud offering whenever possible.

While google cloud offers load balanced HTTP ingress by default it is apparently very expensive in comparison to running small nodes and I have heard only good things about using Traefik for kubernetes ingress.

For setting up Traefik I followed [Manuel's excellent guide](https://medium.com/google-cloud/traefik-on-a-google-kubernetes-engine-cluster-managed-by-terraform-ad871be8ee26) with minor modifications (you can find the final files at the end of the article.)

# HTTPs and Let's encrypt
Traefik has built-in support for automatically getting and renewing HTTPS certificates with [Let's Encrypt](https://letsencrypt.org/). As HTTPS is good practice and a requirement for HTTP2 and PWAs anyway I set it up using [example configurations](https://docs.traefik.io/user-guide/examples/#onhostrule-option-with-http-challenge) from the Traefik docs.

Because I was using just one node for Traefik I chose to go with the easy setup of a local acme.json file that stores the certificate while the node is running.

{{< freelance >}}

# GKE Preemptible nodes, your own chaos monkey
To save costs I chose to use "Preemtible VMs" as nodes to power my kubernetes cluster on GKE. According to google's docs: "Preemptible VMs are Google Compute Engine VM instances that last a maximum of 24 hours and provide no availability guarantees." This means the nodes in my kubernetes cluster randomly go down and are never up more than 24h. While this obviously is not a smart decision for a production setup I have chosen to embrace it and consider the nodes going down my own ["chaos monkey"](https://netflix.github.io/chaosmonkey/) that forces me to write resilient code.

A concrete example I ran into: The Let's encrypt production API has a rate limit of requesting 5 certificates for the same URL in a week. Because my initial naive setup did not save the certificate anywhere it got lost whenever my Traefik node was terminated. While Traefik regenerates the certificate without any issue on startup... after five startups I hit my rate limit and was greeted by an insecure warning without certificate.

# Shared K/V store for Traefik with Zookeeper
Enter a shared Key/Value store for Traefik. Using one is required if you want to run Traefik in cluster mode anyway (and I like to think my setup is easily scalable). It also means I can store my generated certificate in the K/V store where it will no longer just disappear when Traefik restarts.

Since I have previous experience with Zookeeper and the setup was relatively painless I went with it.



# All Kubernetes yaml files for the setup
Finally the meat of the blog post, my complete setup as yaml files you can directly deploy into your GKE cluster:
## Set up Zookeeper first
From this excellent ressource: [https://github.com/kow3ns/kubernetes-zookeeper/blob/master/manifests/README.md](https://github.com/kow3ns/kubernetes-zookeeper/blob/master/manifests/README.md)
```yml
apiVersion: v1
kind: Service
metadata:
  name: zk-hs
  labels:
    app: zk
spec:
  ports:
  - port: 2888
    name: server
  - port: 3888
    name: leader-election
  clusterIP: None
  selector:
    app: zk
---
apiVersion: v1
kind: Service
metadata:
  name: zk-cs
  labels:
    app: zk
spec:
  ports:
  - port: 2181
    name: client
  selector:
    app: zk
---
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: zk
spec:
  serviceName: zk-hs
  replicas: 1
  podManagementPolicy: Parallel
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: zk
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                    - zk
              topologyKey: "kubernetes.io/hostname"
      containers:
      - name: kubernetes-zookeeper
        imagePullPolicy: Always
        image: "gcr.io/google_containers/kubernetes-zookeeper:1.0-3.4.10"
        resources:
          requests:
            memory: "200M"
            cpu: "0.3"
        ports:
        - containerPort: 2181
          name: client
        - containerPort: 2888
          name: server
        - containerPort: 3888
          name: leader-election
        command:
        - sh
        - -c
        - "start-zookeeper \
          --servers=1 \
          --data_dir=/var/lib/zookeeper/data \
          --data_log_dir=/var/lib/zookeeper/data/log \
          --conf_dir=/opt/zookeeper/conf \
          --client_port=2181 \
          --election_port=3888 \
          --server_port=2888 \
          --tick_time=2000 \
          --init_limit=10 \
          --sync_limit=5 \
          --heap=512M \
          --max_client_cnxns=60 \
          --snap_retain_count=3 \
          --purge_interval=12 \
          --max_session_timeout=40000 \
          --min_session_timeout=4000 \
          --log_level=INFO"
        readinessProbe:
          exec:
            command:
            - sh
            - -c
            - "zookeeper-ready 2181"
          initialDelaySeconds: 10
          timeoutSeconds: 5
        livenessProbe:
          exec:
            command:
            - sh
            - -c
            - "zookeeper-ready 2181"
          initialDelaySeconds: 10
          timeoutSeconds: 5
        volumeMounts:
        - name: datadir
          mountPath: /var/lib/zookeeper
      securityContext:
        runAsUser: 1000
        fsGroup: 1000
  volumeClaimTemplates:
  - metadata:
      name: datadir
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 5Gi
```

## Permissions for Traefik
```yml
# create Traefik cluster role
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
 name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
---
# create Traefik service account
kind: ServiceAccount
apiVersion: v1
metadata:
  name: traefik-ingress-controller
  namespace: default
---
# bind role with service account
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: default
```

## Traefik config
Note the configuration of zookeeper using the service address for the "client service" (cs) as well as the Let's encrypt config here.
```yml
# define Traefik configuration
kind: ConfigMap
apiVersion: v1
metadata:
  name: traefik-config
data:
  traefik.toml: |
    # traefik.toml
    defaultEntryPoints = ["http", "https"]
    [entryPoints]
      [entryPoints.http]
        address = ":80"
        [entryPoints.http.redirect]
          entryPoint = "https"
      [entryPoints.https]
      address = ":443"
        [entryPoints.https.tls]

      [zookeeper]
        endpoint = "zk-cs.default.svc.cluster.local:2181"
        watch = true
        prefix = "traefik"

      [acme]
      email = "your@email.com"
      storage = "traefik/acme/account"
      onHostRule = true
      caServer = "https://acme-v02.api.letsencrypt.org/directory"
      acmeLogging = true
      entryPoint = "https"
        [acme.httpChallenge]
        entryPoint = "http"

      [[acme.domains]]
        main = "your.domain.com"
```

## Deployment for Traefik
I run just one replica in here to save costs in my dev setup but I've also scaled it up to three to test if it would stay up 100% of the time even with random nodes going down and everything works fine :).
```yml
# declare Traefik deployment
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: traefik-ingress-controller
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      volumes:
        - name: config
          configMap:
            name: traefik-config
      containers:
      - name: traefik
        image: "traefik:1.7.14"
        volumeMounts:
          - mountPath: "/etc/traefik/config"
            name: config
        args:
        - --configfile=/etc/traefik/config/traefik.toml
        - --kubernetes
        - --logLevel=INFO
```

## Traefik service
```yml
# Declare Traefik ingress service
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-controller
spec:
  selector:
    app: traefik-ingress-controller
  ports:
    - port: 80
      name: http
    - port: 443
      name: tls
  type: LoadBalancer
```

{{< freelance >}}

# Final result
The final workloads with traefik and zookeeper

![Traefik and Zookeeper workloads](/img/posts/run-a-personal-cloud-with-traefik-lets-encrypt-and-zookeeper/workloads.png#center)

And the kubernetes ingresses (ignore the app I used as demo for this).

![Kubernetes ingresses](/img/posts/run-a-personal-cloud-with-traefik-lets-encrypt-and-zookeeper/ingress.png#center)

{{< aboutme >}}