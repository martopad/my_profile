+++
title = 'First_app_on_k8'
date = 2024-08-18T20:00:52+02:00
toc = true
+++

## Goals
1. Put my Kubernetes cluster to use.
    * I have many things planned, but deploying a simple,
    stateless, and static website is a good starting point.
2. Learn how to expose Kubernetes services.
3. Learn how to expose endpoints to the public internet.
    * Without an ISP-provided public IP.
4. Learn about the workflow of web development.
    * Most of my development experience focuses on embedded and Linux native.

## Setup
1. Use multiple istio ingress controllers to assign to different
web services.
2. Use multiple metallb IP pools to assign IPs to these ingress
controllers.
3. Hugo as framework/tool so that I only have to
think about the content.
4. Cloudflare Tunnels to expose my endpoint.
5. GlusterFS as network storage backend to store the website files.

## Journey
To my surprise, Kubernetes is magical to use after provisioning a Kubernetes
cluster. Though it still triggers my
*this-is-too-easy-I-must-be-doing-something-wrong* senses. Establishing
redundancy and resiliency is easy. The challenge here is connecting these
services and creating a workflow for development and deployment.

So, I started by figuring out how to use an ingress controller, which is
the first step in accessing web services outside the cluster. I followed
[this tutorial](https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/).
The tutorial gave me the idea that after installing Istio,
a gateway associated with my basic httpd application needs to be created.

After that, get an IP address to access the service. Then comes MetalLB.
I was looking at their [documentation](https://metallb.io/usage/).
Configuring these two enabled me to access a basic Httpbin website using a private IP.

Then, to expose this publicly, I used a service called Cloudflare Tunnels.
Cloudflare Tunnels is a convenient and free tool that creates an HTTP-based
tunnel between my private K8s cluster and their magic servers.
Their setup is pretty simple. I followed these tutorials:

1. [Basic non-K8s deployment](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-local-tunnel/)
2. [K8s + cloudflared basics](https://developers.cloudflare.com/cloudflare-one/tutorials/many-cfd-one-tunnel/)
3. [K8s + cloudflared more details](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/deploy-tunnels/deployment-guides/kubernetes/#_top)

After reading more Kubernetes documentation, I managed to have an
end-to-end web app exposed to the internet.

Currently, my proof-of-concept is running on default settings. I've been
following the documentation, copying and pasting example code to see if
it runs as described. However, I've had to change these deployments
to suit my needs to understand and learn genuinely.

Things that need to change:
1. Create two static websites.
    * 1 for my CV
    * 1 for my profile/blog/more detailed stuff.
2. Deploy two ingress controllers. One for each static site.
3. Customize MetalLB to control what IP gets allocated to each ingress controller.
4. Modify the Cloudflare manifest to route traffic depending on which hostname is proper.

*Note: The code included here are abbreviated versions of what I deployed
and will not work when copy-pasted. You can look at this
[Github Repo](https://github.com/martopad/k8s-deployments/tree/main)*
if you wan the working code.

Number one was manageable. However, I had to use bind mounts because Kubernetes
removed GlusterFS support. I used Nginx and just bind mounted where the location
of the static site files are:
```yaml 
#app.yaml
...
        volumeMounts:
          - mountPath: /usr/share/nginx/html
            name: website-source-cv
            readOnly: true
      volumes:
      - name: website-source-cv
        hostPath:
          path: '/mnt/gv0/production/websites/cv.martinlutheryung.com/html'
          type: Directory
```
Although there are security concerns with node-level bind mounts, this is a good compromise.

As for number 2, Istio provides a way to ovveride default installation using
their installer, [here](khttps://istio.io/latest/docs/setup/additional-setup/customize-installation/).

I then looked at the default profile inside the installer-- `istio/manifests/profiles/default.yaml`:

FROM:
```yaml
...
    # Istio Gateway feature
    ingressGateways:
    - name: istio-ingressgateway
      enabled: true
    egressGateways:
    - name: istio-egressgateway
      enabled: false
```

TO:
```yaml
    ingressGateways:
    - name: istio-ingressgateway
      enabled: true
    - name: istio-ingressgateway-profile
      enabled: true
      k8s:
        serviceAnnotations:
          metallb.universe.tf/address-pool:  ipprofilemly
        overlays: # This object will modify default properties of the component
            - kind: HorizontalPodAutoscaler
              name: istio-ingressgateway-profile
              patches:
                - path: metadata.labels.app
                  value: istio-ingressgateway-profile
                - path: metadata.labels.istio
                  value: ingressgateway-profile
                - path: spec.scaleTargetRef.name
                  value: istio-ingressgateway-profile
            - kind: Deployment
              name: istio-ingressgateway-profile
              patches:
                - path: metadata.labels.app
                  value: istio-ingressgateway-profile  # Change the label to istio-ingressgateway-profile
                - path: metadata.labels.istio
                  value: ingressgateway-profile
                - path: spec.selector.matchLabels.app
                  value: istio-ingressgateway-profile
                - path: spec.selector.matchLabels.istio
                  value: ingressgateway-profile
                - path: spec.template.metadata.labels.app
                  value: istio-ingressgateway-profile
                - path: spec.template.metadata.labels.istio
                  value: ingressgateway-profile
            - kind: Service
              name: istio-ingressgateway-profile
              patches:
                - path: metadata.labels.app
                  value: istio-ingressgateway-profile
                - path: metadata.labels.istio
                  value: ingressgateway-profile
                - path: spec.selector.app
                  value: istio-ingressgateway-profile
                - path: spec.selector.istio
                  value: ingressgateway-profile
#... did the same for ingressgateway-cv
    egressGateways:
    - name: istio-egressgateway
      enabled: false
    - name: istio-egressgateway-profile
      enabled: false
    - name: istio-egressgateway-cv
      enabled: false
```

The change was significant because, without it, MetalLB could not detect which
ingress controller to give the proper IP. Here is the IP pool configuration.
```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: ipprofilemly
  namespace: metallb-system
spec:
  addresses:
    - 192.168.1.144/32
  autoAssign: false
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: ippool
  namespace: metallb-system
spec:
  ipAddressPools:
  - ipprofilemly
```

I wanted to control the IP address assigned to each ingress controller to prevent
possible breakage when MetalLB automatically assigns a different IP address.

Once deployed. It will look like this:
```bash
istio-system     service/istio-ingressgateway-profile   LoadBalancer   10.96.16.81     192.168.1.144   15021:31023/TCP,80:32635/TCP,443:30120/TCP   5d21h
```

This setup is not locally reacheable because the there is no gateway that connects
the ingress controller and the app's pods/containers. This is where the istio gateway and virtual
service come in.

```yaml
#gateway.yaml
piVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: profilemly-gateway
spec:
  selector:
    app: istio-ingressgateway-profile
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "profile.martinlutheryung.com"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: profilemly
spec:
  hosts:
  - "profile.martinlutheryung.com"
  gateways:
  - profilemly-gateway
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        port:
          number: 80
        host: profilemly
```

It is now possible to curl this endpoint inside my local network.
```bash
curl -s -I -HHost:profile.martinlutheryung.com http://192.168.1.144:80
HTTP/1.1 200 OK
server: istio-envoy
date: Sat, 24 Aug 2024 14:13:31 GMT
content-type: text/html
content-length: 3674
last-modified: Sun, 18 Aug 2024 20:15:19 GMT
etag: "66c25657-e5a"
accept-ranges: bytes
x-envoy-upstream-service-time: 6
```

Then finally is to configure the cloudflare public endpoint.

This was easy to setup, since the cloudflared service can route to
different hostname by just creating a config map:


```yaml
# Note: There are other configuration done asides from this configMap.
#       But it was all left to the defaults as told in the documentation.
...
apiVersion: v1
kind: ConfigMap
metadata:
  name: cloudflared
data:
  config.yaml: |
    tunnel: kubernetes
    credentials-file: /etc/cloudflared/creds/credentials.json
...
    ingress:
    # The first rule proxies traffic to the httpbin sample Service defined in app.yaml
    - hostname: profile.martinlutheryung.com
      service: http://192.168.1.144:80
    - hostname: cv.martinlutheryung.com
      service: http://192.168.1.145:80
```
The cloudflared service just needs to route the requests to to proper Istio
ingress controllers. Then viola! It works!
