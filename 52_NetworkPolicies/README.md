# NetworkPolicies

So far we have seen that every pod is assigned it's own IP address and all the pods in a cluster can reach each other using these IP addresses, and consequently via services too. That's not something that you might want, e.g. a wordress pod usng a mysql pod as it's database, in that scenario, only the wordpress pods needs access to the mysql pod.

So to fix this we need to set up firewalls in our kubernetes networking, and that's done by creating NetworkPolicy objects. 







## References

[https://kubernetes.io/docs/concepts/services-networking/network-policies/](https://kubernetes.io/docs/concepts/services-networking/network-policies/)