Here's a simple CPU + memory stress combo one-liner:

sh -c 'for i in $(seq 1 8); do while :; do :; done & done; tail /dev/zero | head -c 500M > /dev/null & sleep 60'


This:

    Spins 8 CPU loops

    Streams 500MB of memory junk to /dev/null (allocating memory in the process)
    
    
    