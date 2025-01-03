
After various testing and validation, based on the graph below 

* we can see that Max peak bandwidth is 3.122 Mb/s which is around 25Mbps 
* we can see that Min peak bandwidth is about 0.022 Mb/s which is around 0.176Mbps
* we can see that the Mean peak bandwidth is 0.217 Mb/s which is around 1.736Mbps

Based on the results from this cluster and other cluster tested against, I have set the values for Max and Min bandwidths to the values below with some buffer for Max bandwidth.

```yaml
      maxBandwidthMbps: 40
      minBandwidthMbps: 10