+++
title = 'First_app_on_k8'
date = 2024-08-18T20:00:52+02:00
toc = true
+++

## Goals
1. Put my Kubernetes cluster to use.
    * I have many things planned for it but deploying the
    a simple, stateless, and static website is a good starting
    point. 
2. Learn how to expose Kubernetes services.
3. Learn how to expose endpoints to the public internet.
    * Without a public IP.
4. Learn a little bit of workflow of web development.
    * My software development experience is geared towards
    embedded, and native.

## Setup
1. Use multiple istio ingress controllers to assign to different
web services.
2. Use multiple metallb ip pools to assign IPs to these ingress
controllers.
3. Hugo as framework/tool so that I only have to
think about the content.
4. Cloudflare tunnels to expose my endpoint.
5. GlusterFS as network storage backend to store the website files.

## Journey
To my surprise, once the Kubernetes cluster is provisioned, Kubernetes
is magical to use. Though it still triggers my
*this-is-too-easy-I-must-be-doing-something-wrong* senses. Estabishing
redundancy, and resiliency is easy. The challenge here is connecting
these services, and creating a workflow for development to deployment.

So I started with figuring out to use an ingress controller, as that is
the first step in order to access webservices outside of the cluster. I
followed this[tutorial](https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/).
This gave me the basic idea that after installing istio a gateway needs
to be created that is associated with my basic httpd application.

After that, I immediately was pointed to my next hurdle, how do I get
an IP address that I can access the service. Then comes metallb. Looking
at their [documentation](https://metallb.io/usage/).

Configuring these two, yielded me being able to access a basic httpbin
website using a private IP.

Then to expose this publicly, I used a service called Cloudflare Tunnels.
This is a convenient, and free, tool that creates a HTTP-based tunnel
between my private K8s cluster and their magic servers. Their setup is
fairly simple. I followed these tutorials:
1. [Basic non-K8s deployment](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-local-tunnel/)
2. [K8s + cloudflared basics](https://developers.cloudflare.com/cloudflare-one/tutorials/many-cfd-one-tunnel/)
3. [K8s + cloudflared more details](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/deploy-tunnels/deployment-guides/kubernetes/#_top)

After some more Kubernetes documentation reading I managed to have an
end-to-end webapp that is exposed to the internet.

As of now, this proof-of-concept are all running just the defaults. I just
read their documentations and copy pasted the example code and see if
it runs as described. To truly learn, I needed to change these deployments
to suite my needs.

Things that needs to change:
1. Create two static website.
    * 1 for my cv
    * 1 for my profile/blog/more detailed stuffs.
2. Deploy 2 ingress controllers. One for
each static site.
3. Customize metallb to have control on what IP gets
allocated to each ingress controller.
4. Modify cloudflared manifest to properly route
traffic depending on which hostname.

[Add more]

