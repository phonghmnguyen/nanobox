## High Level Design

![cluster (1)](https://github.com/user-attachments/assets/720f44e2-ed22-4ffa-ade2-c1eb6ed4f27b)



## Challenges during implementation:
- Sync lru evictions
  + Read requests on replicas will update their lru independently of one another and the master. When capacity is reached, every node will likely have different sets of evicted keys which leads to divergence
  + One solution could be to disable eviction on replicas and sync evictions from the master with delete request to replicas. This also means that the lru property is only guaranteed when read is forwarded to the master. With that said, we can only provide an approximation of the lru property for performance and consistency reasons. However, arguably if reads for a key happen on both master and replica nodes, the key is a hot key, and since reads should be uniformly distributed on all nodes, this shouldn't be a problem as a hot key is very unlikely to be evicted by the master. Wrong evictions of cold keys shouldn't be a big problem if we can gain significant performance in return.
  + This also comes with some limitations.
    1. Read requests to replicas can return evicted entries due to delay. In any case, reads are eventually consistentent if enabled on replicas
    2. Master won't know about key access statistics of replicas and so it won't be able update its lru accordingly which is very costly anyway


## Planning:
- Implement partitions with consistent hashing, each shard will be managed by its own raft group
- Implement Kubernetes operator
- Improve telemetry


  
