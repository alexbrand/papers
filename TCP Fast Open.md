# TCP Fast Open

http://static.googleusercontent.com/media/research.google.com/en/us/pubs/archive/37517.pdf

Sivasankar Radhakrishnan, Yuchung Cheng, Jerry Chu, Arvind Jain, Barath Raghavan

## Summary
The authors propose a new TCP mechanism called TCP Fast Open (TFO), which allows the exchange of data during the TCP handshake.
The motivation behind this new mechanism is the fact that about 10%-30% of a web request's latency is caused by the TCP handshake.
Exchanging data during the handshake would reduce the latency by one round-trip time (RTT).
