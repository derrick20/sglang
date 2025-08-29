---
{"dg-publish":true,"permalink":"/work/sg-lang/req-to-token-pool/"}
---


Function:
(req, seq_pos) -> cache_loc

These cache_loc are used by [[Work/SGLang/TokenToKVPool\|TokenToKVPool]]
Maps requests to a TokenPool

They describe here how the memory pool works:


```python title:memory_pool.py

class ReqToTokenPool:
	self.size
	self.max_context_len
	self.device
	self.req_to_token = torch.zeros((size, max_context_len), device=device)
```