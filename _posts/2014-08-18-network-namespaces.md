---
title: Network Namespaces and Traffic Control
layout: post
---

Well I fixed up this blog a bit after I broke it back in February so I could post about some cool stuff that I've been doing for work.

Our application is still under wraps right now, so I can't talk too much about it or about the shape that this task took when it got into our proprietary code, but suffice to say that I wanted to test what happened when the network went bad when trying to communicate with a service elsewhere in the cloud, without spinning up a new EC2 instance (so that this test could be automated to be fast and isolated from real networks).

I found some okay guides for setting up a network namespace on Google but they all stopped, irritatingly, right when it became time to make the namespace talk to the outside world. Most said something like "now just set up a bridge!" which turned out to be very difficult (it might not be possible at all, but I'm unsure) on EC2.

So first let's talk about what a network namespace is. A network namespace is a feature provided by the `ip-netns` tool (which is run as `ip netns` but looked up as `man ip-netns` just to confuse everyone) which is part of the seemingly-poorly-understood `ip` suite of tools that was supposed to deprecate `ifconfig`, `route`, et al seven years ago, but for whatever reason most of the Linux community is not only still using `ifconfig` but is teaching newcomers to use it. Anyway that's not the point.

Functionally, `ip netns` is a feature that allows us to set up isolated, private virtual subnets on one host, and connect them to each other with virtual ethernet devices, to simulate a larger LAN on a single host.

If you've used Docker or LXC (Linux Containers), this should seem familiar. Docker uses `ip-netns` in their implementation of containers, or rather, they use LXCs to build Docker, which in turn use `ip-netns`.

In my case I wanted to set up these namespaces to allow me to degrade the connection from one service to another using the `tc` tool so I could see how these services behave when the network is misbehaving.

So first we want to create a new namespace:

{% highlight bash %}
$ sudo ip netns add blue
{% endhighlight %}

This creates a namespace called `blue` on the current system. View it by doing

{% highlight bash %}
$ sudo ip netns ls
blue
{% endhighlight %}

Alright that's nice, now it exists, so let's create two virtual ethernet devices that are paired together, one inside of the `blue` namespace and one in the root namespace.

{% highlight bash %}
$ sudo ip link add veth0 type veth peer name veth1
{% endhighlight %}

This creates `veth0` and `veth1`. Let's put `veth1` into `blue` the namespace:

{% highlight bash %}
$ sudo ip link set veth1 netns blue
{% endhighlight %}

Now if you do `ip addr` directly on the command line you'll see `veth0` but `veth1` appears to have disappeared! If we use the `ip netns exec [NAMESPACE] [COMMAND]` command, then COMMAND will be executed inside of the new network namespace!

Notice that calling `sudo ip netns exec blue ip addr` will show us the network status inside of the namespace right now:

{% highlight bash %}
$ sudo ip netns exec blue ip_addr
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
4: veth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 8e:a9:0d:2f:7c:d8 brd ff:ff:ff:ff:ff:ff
{% endhighlight %}

OK so let's add IP addresses using `ip addr` both inside and outside of the namespace, and bring up our three devices (`veth1`, `veth0`, and `lo` (inside `blue`))

{% highlight bash %}
$ sudo ip addr add 10.1.1.0/31 dev veth0
$ sudo ip netns exec blue ip addr add 10.1.1.1/31 dev veth1
$ sudo ip link set veth0 up
$ sudo ip netns exec blue ip link set veth0 up
$ sudo ip netns exec blue ip link set lo up
{% endhighlight %}

I chose a `/31` so we only consume two IPs, and obviously used private `10.` IP addresses. Then set up a route for traffic inside the namespace to reach the outer world:

{% highlight bash %}
$ sudo ip netns exec blue ip route add default via 10.1.1.0
{% endhighlight %}

And that's it for the `netns` stuff, and that's where most howtos stop. But you'll see that although the root namespace can see into the `blue` namespace, and vice versa, processes inside the namespace cannot see the outer world:

{% highlight bash %}
$ ping 10.1.1.1
PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
64 bytes from 10.1.1.1: icmp_seq=1 ttl=64 time=0.069 ms
64 bytes from 10.1.1.1: icmp_seq=2 ttl=64 time=0.046 ms
^C
--- 10.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.046/0.057/0.069/0.013 ms
$ sudo ip netns exec blue ping 10.1.1.0
PING 10.1.1.0 (10.1.1.0) 56(84) bytes of data.
64 bytes from 10.1.1.0: icmp_seq=1 ttl=64 time=0.049 ms
64 bytes from 10.1.1.0: icmp_seq=2 ttl=64 time=0.074 ms
64 bytes from 10.1.1.0: icmp_seq=3 ttl=64 time=0.051 ms
^C
--- 10.1.1.0 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1998ms
rtt min/avg/max/mdev = 0.049/0.058/0.074/0.011 ms
$ sudo ip netns exec blue ping google.com
ping: unknown host google.com
{% endhighlight %}

While this might be OK for some setups, it wasn't OK for me, so after being unable to figure out how to get the process inside to see the outside world, I reached out to the Hacker School network and was aided by one [Andrew Mulholland](http://twitter.com/itwasntandy) who helped me by creating an appropriate IPTables rule to set up NAT, and that's the following:

{% highlight bash %}
$ sudo iptables -t nat -A POSTROUTING -s 10.1.1.0/31 -d 0.0.0.0/0 -j MASQUERADE
{% endhighlight %}

This rule tells `iptables` to add the source `10.1.1.0/31` to the NAT table, to intercept any traffic headed to `0.0.0.0/0` (any destination) and to `MASQUERADE` that traffic, ie, route it like your router.

Additionally we must allow NAT on the system:

{% highlight bash %}
$ sudo sysctl net.ipv4.ip_forward=1
{% endhighlight %}

And now, through magic, we can see Google!

{% highlight bash %}
$ sudo ip netns exec blue ping google.com
PING google.com (173.194.115.66) 56(84) bytes of data.
64 bytes from dfw06s41-in-f2.1e100.net (173.194.115.66): icmp_seq=1 ttl=56 time=15.2 ms
64 bytes from dfw06s41-in-f2.1e100.net (173.194.115.66): icmp_seq=2 ttl=56 time=15.9 ms
^C
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 15.240/15.576/15.912/0.336 ms
{% endhighlight %}


Finally, the point of all this, was so that we can test what happens when network connectivity is bad. Let's tell the kernel to drop 30% of the packets that are sent through `veth1` to simulate a "bad network" and then use ping to see the new packet queueing strategy in action. We do this with the `tc` command, which stands for "traffic control" and more information on this command and on reshaping network can be seen by running `man tc`, `man netem`, and also on [this helpful website](http://www.linuxfoundation.org/collaborate/workgroups/networking/netem).

{% highlight bash %}
$ sudo ip netns exec blue tc qdisc add dev veth1 root netem loss 30%
{% endhighlight %}

Now we can see the effect of this simply by `ping`ing `10.1.1.1` from inside of the root namespace:

{% highlight bash %}
$ ping 10.1.1.1
PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
64 bytes from 10.1.1.1: icmp_seq=3 ttl=64 time=0.082 ms
64 bytes from 10.1.1.1: icmp_seq=4 ttl=64 time=0.077 ms
64 bytes from 10.1.1.1: icmp_seq=5 ttl=64 time=0.073 ms
64 bytes from 10.1.1.1: icmp_seq=6 ttl=64 time=0.074 ms
64 bytes from 10.1.1.1: icmp_seq=7 ttl=64 time=0.077 ms
64 bytes from 10.1.1.1: icmp_seq=10 ttl=64 time=0.081 ms
^C
--- 10.1.1.1 ping statistics ---
11 packets transmitted, 6 received, 45% packet loss, time 9996ms
rtt min/avg/max/mdev = 0.073/0.077/0.082/0.007 ms
{% endhighlight bash %}

Finally, you can see that due to the laws of randomness our actual dropped packet rate was a little higher than we requested, but this will converge on the requested rate as `n` packets goes to infinity.

I hope this little walkthrough on setting up network namespaces, allowing the new namespace to see the outside world through NAT, and altering the traffic control rules for the isolated namespace are helpful to someone out there, and/or are inspiration for more complicated setups and traffic shaping rules.

I think this stuff is a lot of fun, one way or the other!
