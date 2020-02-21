---
title: "First impressions on Kubernetes"
date: 2020-02-21T17:48:50-03:00
draft: false
---

I have deployed two simple apps on Google's managed version of Kubernetes,
Google Kubernetes Engine. This isn't enough for me to judge the whole thing,
obviously, but I would like to write down my impressions.

First of all, there is a very "simple" process. You should create your
deployment manifest - which is, mostly, a file describing what your app should
look like in terms of ports it's listening to, volumes it needs, environment
variables, secrets, etc. If your app is Dockerized, then this should be enough
for you to have a working version of your app. Commands like `kubectl
port-forward` can be your friend by forwarding the port in which your app is
running in the pod - a pod is a collection of containers, most of the times
it'll be a single container pod but, sometimes, you'll have sidecar containers 
(like a proxy to access your Google Cloud SQL database). You can then access
`localhost:app-port` to easily test it. I quite liked the way we can describe
our app, it seems like we are creating a docker-compose file, in a way.
Kubernetes forces you to deploy your app in a containerized manner.

After creating your deployment, if you want to access your app internally or
externally, you'll have to create a "service" to define how the app will be
accessible, if TLS will be enabled, which port forwards to which port, maybe
your app will be accessed through a hostname (like app.tretinha.com, etc.).

Something I struggled a little bit with was in getting to know the differences 
between the types of services, why most of the times an ingress resource is 
just better than creating a Load Balancer and exposing it's ports to the 
world. I thought that ingresses and services were two different things and, in 
fact, I was somewhat right. An ingress resource is a complementary part of a 
service, you can create a NodePort traffic that describes the app's ports and, 
then, use an ingress to forward traffic to this service. An interesting thing 
is that GKE comes packed with an ingress controller by default, the GCE 
controller, which means you don't always have to use a more robust controller 
like NGINX. It has a few downsides like not being able to redirect a request, 
and having its health checks on "/" (if you have, for example, a login screen 
that redirects every user that's not logged in, the health checks will make 
your containers/pods go to an unhealthy state, screwing your deploy - that's 
manageable by creating a readinessProbe at the "/login" path, though). You 
have to wait a little bit for your ingress to work, it's not always something 
fast.

A very good thing is that you can limit the sizes of your container's machine, 
which doesn't cut you loose on the control part in comparison to a regular
virtual machine. But the debugging part, in the first days, was infuriating.
Not having an easy way to access your failed containers for debugging (I don't
want logs only, for satan's sakes, I want to be able to run commands on the
machine) is a very frustrating experience; your failed containers just don't
exist anymore when a deploy fails, which means you can't access an unexistent
machine. I didn't completely overcome this until now, I just got better at
describing my services and deployments and didn't need that much to go into the
machines. That's a bummer.

Looking in the perspective of a company, it's very good to use Kubernetes
because it's very easy to understand how to deploy the app. The knowledge gets
more interchangeable. If you get this aligned with an infrastructure management
tool like Terraform, you'll basically have your whole cloud infrastructure
"described".

I'll probably create more articles on Kubernetes since the journey is only
starting (maybe rectify a few things too? heh).
