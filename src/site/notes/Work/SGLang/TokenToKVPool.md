---
{"dg-publish":true,"permalink":"/work/sg-lang/token-to-kv-pool/"}
---

Function:
(layer, cache_loc, head, head_dim) -> cache_k, cache_v


- The idea is we want to consider all the heads and batch sequences in parallel. Each one accesses the element which 
	- The position represents a singular token's collection of cache_k and cache_v (floats).
	- We have H head_dim vectors of values
	- <H, head_dim>


```python title:mem_cache/memory_pool.py

class ReqToTokenPool:
	def alloc(self, need_size):
	def free(self, free_index):
```