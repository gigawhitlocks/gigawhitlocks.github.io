---
title: Implementing Ping and Traceroute
layout: post
---

Over the last couple of days a fellow Hacker Schooler (Martin Törnwall)[http://github.com/mtornwall] and I implemented [`ping` and `traceroute`](https://github.com/thewhitlockian/icmp-utils) for fun and to learn more about icmp and how these utilities work.

One of the surprising things that we discovered early on is that `ping` actually requires root access to run on a Linux machine. This is surprising because, of course, the executable is available to unprivileged users, but the different distributions handle granting this access differently. Mine uses "capabilities" to grant the executable the ability to open raw sockets explicitly and no other permissions. Older distributions use a different method.

We wrote the programs in C so that we could try our hand at the low-level end of things, and because I wanted to work with Martin to get better at C, since he's a wizard.

`ping` is a pretty simple program, but turned out not only to be the bulk of the work but `traceroute` wound up depending on it, and I was able to reuse a lot of the code that was written for `ping` when I wrote `traceroute`. In fact, despite originally setting out to write `traceroute` and expecting it to take longer, `ping` took about six hours and `traceroute` only took about two, once we had `icmp-utils.c` and `icmp-utils.h` completed. The only real difference between `ping` and `traceroute` is the TTL, or "time to live", setting in the header of the ICMP packet. C provides `setsockopt()` which allowed me to easily set the TTL on the outgoing packet.

Since this was a quick-and-dirty implementation, I was satisfied by having the program `ping` the target first, to determine the destination. Then, successively incrementing the TTL, I was able to see each "hop" as it occurs because the packet would reach its TTL and the current hop would send a reply to notify my program that the packet had died. That hop would, of course, include its IP in the packet, and thus we're able to trace the route to the destination.

The biggest challenge was calculating the checksum for the outgoing ICMP packets, and we had to use Wireshark to watch incoming packets to debug this, as we did not get it on the first try. Creating the checksum involved doing a ones-complement sum and I was happy to have Martin expedite the process. It's something I could have done but would have taken me a bit longer than that wizard, and I learned a number of tricks as he took over during that part of the project and guided me through what he was doing. These experiences are really what make Hacker School unique.

The code for that section is here:
{% highlight c %}
uint32_t sum = 0;
int i;
for (i = 0; i < sizeof(message)/2; i++) { 
  //this assumes even message size
  sum += *((uint16_t *)&message+i);
}

// Finish calculating and set message checksum
message.icmp_cksum = (uint16_t) ~(sum + (sum >> 16));
{% endhighlight %}

`message` in this case is the outgoing packet. `sum` is a 32-bit unsigned int because we needed to add up the value of each byte in `message` and capture the overflow in the upper bits, to be then added `message.icmp_cksum`, which is a 16-bit unsigned int (hence the bit-shift to allow us to add in the overflow).

This wasn't the biggest project I've tackled at Hacker School, but it's certainly one of my favorites.
