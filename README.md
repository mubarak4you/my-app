etoke-306

Optimize cost and performance - grafana

Memory

The graph here shows the baseline memory usage of the grafana pod over the last hour, the memory usage appears relatively stable at around 74 MiB until the Total pod memory usage and Pod CPU usage (mcores) dashboards are opened on the browser. 

￼

A stress test was then performed with a script that simply sends http request to a specific grafana dashboard using curl. 
The script requires grafana admin credentials to authentic .

Sequential request and Concurrent request are both set to 25.

Sequential request sends 25 requests to the dashboards one after the other
(One request at.a time no overlapping)
which then triggers the dashboard to make a request to Prometheus by scraping the metrics. 

Concurrent request sends 25 of the same request to the dashboard till the sequential request is met which is 5 times.
(Overlapping requests, simulating multiple users accessing the dashboard simultaneously) 
which then triggers the dashboard to make a request to Prometheus by scraping the metrics. 
  Carried out another test
Sequential request and Concurrent request are both set to 5.

Sequential request sends 5 requests to the dashboards one after the other
(One request at.a time no overlapping)
which then triggers the dashboard to make a request to Prometheus by scraping the metrics. 

Concurrent request sends 5 of the same request to the dashboard till the sequential request is met which is 5 times.
(Overlapping requests, simulating multiple users accessing the dashboard simultaneously) 
which then triggers the dashboard to make a request to Prometheus by scraping the metrics.   
Carried out another test
By manually opening 5 different dashboard pages on the browser and refreshed each pages to trigger the dashboards being loaded.

￼

Results
Based on the various stress test we can see the spikes on the Memory usage not go over 108 MiB, so from the result of the test we can set the Memory Limit for the Grafana pod to about 130 MiB and Memory request to 85 MiB, giving a buffer for load and unexpected spikes.

Setting the Memory Request to 
    resources:
      requests:
        memory: “80Mi”


Setting the Memory Limit to 
    resources:
      limits:
        memory: “130Mi”



Cpu

The graph here shows the baseline CPU usage of the grafana pod over the last hour, the CPU usage appears relatively stable at around 1.5mcores until the the Total pod memory usage and Pod CPU usage (mcores) dashboards are opened on the browser. 

￼

Results
Based on the same stress test we can see the spikes on the CPU with the CPU usage not go over 5.5mcores, so from the result of the test we can set the CPU Limit for the Grafana pod to about 7mcores and CPU request to 3mcores, giving a buffer for load and unexpected spikes.
 
￼

Based on the graph after the stress test.

Setting the CPU Request to 
    resources:
      requests:
        cpu: “8m"


Setting the CPU Limit to 
    resources:
      limits:
        cpu: “20m"


Mem and CPU 
    resources:
      limits:
        cpu: “20m"
        memory: “130Mi"
      requests:
        cpu: “8m"
        memory: “80Mi"

After setting the values above in the grafana configuration and created a new cluster, I noticed the grafana webpage and dashboard taking much longer to load up so I had to bump up the mem and cpu.

    resources:
      limits:
        cpu: 60m
        memory: 180Mi
      requests:
        cpu: 40m
        memory: 100Mi
    persistence:
