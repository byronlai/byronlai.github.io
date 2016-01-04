---
layout: post
title:  "Load Balancing and Automatic Failover with nginx and keepalived"
date:   2013-09-20 09:20:33 +0800
categories: jekyll update
---
When your website does not have many visitors, maybe all you need is a just single server. However, when your business grows, you start to need more servers to handle the traffic. Now we need a way to distribute workloads across the machines. This is the job of a load balancer. In this post, I will walkthrough the steps to setup [nginx](http://nginx.org) as a load balancer. We choose nginx because of its high concurrency, high performance and low memory usage.

First we need to define the upstream servers. These are the machines that will actually handle the requests from our users. In the following configuration, we define two servers at `node1:3000` and `node2:3000`.

{% highlight nginx %}
upstream backend {
    server node1:3000;
    server node2:3000;
}
{% endhighlight %}

Then, we instruct nginx to forward the requests to our backend. After this, we can restart nginx so that the new configuration takes effect.

{% highlight nginx %}
server {
    listen 80;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_next_upstream error http_404;
    }
}
{% endhighlight %}

Now we have a load balancer. However, the load balancer is a single point of failure. In case there is a hardware failure in the server which the load balancer resides, our service will be down. The solution is to run two instances of load balancers. If one fails, the other takes over. This is called automatic failover.

To do that, we will make use of [keepalived](http://www.keepalived.org). Each of our server has its private IP. In addition, there is a virtual IP. Initially, the virtual IP is assigned to the master load balancer. When the master load balancer is down, keepalived will re-assign the virtual IP to the backup load balancer. Our clients will make requests through the virtual IP.

Suppose the virtual IP is 192.168.1.100. The following is the keepalived configuration for the master load balancer.

{% highlight text %}
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass PASS
    }
    virtual_ipaddress {
        192.168.1.100
    }
}
{% endhighlight %}

The following is the keepalived configuration for the slave load balancer. The only difference between the master and slave is the priority.

{% highlight text %}
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass PASS
    }
    virtual_ipaddress {
        192.168.1.100
    }
}
{% endhighlight %}

This is all we need to setup load balancing and automatic failover.
