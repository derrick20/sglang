---
{"dg-publish":true,"permalink":"/work/sg-lang/model-runner/"}
---



self.server_args: ServerArgs
self.req_to_token_pool: [[Work/SGLang/ReqToTokenPool\|ReqToTokenPool]] or [[DecodeReqToTokenPool\|DecodeReqToTokenPool]]
self.token_to_kv_pool_allocator: [[BaseTokenToKVPoolAllocator\|BaseTokenToKVPoolAllocator]]

self.model_config
self.devices

self.sampler
self.model: nn.Module

Memory management here:
- [[Work/SGLang/ReqToTokenPool\|ReqToTokenPool]] maps request to its tokens, storing a tensor size (batch_size, max_context_length)
- [[BaseTokenToKVPoolAllocator\|BaseTokenToKVPoolAllocator]]s manage indices going to kv cache data.
- [[Molecules/KVCache\|KVCache]] holds the cached values:
	- Storing for each head and layer, the ongoing accumulated position's projected key and value.
	- 
```python
class ModelRunner:
	def load_model():
		self.model = get_model(self.model_config, ...)

	def forward(forward_batch: ForwardBatch):
		if forward_batch.forward_mode.is_decode():
			ret = self.forward_decode(forward_batch)
		elif forward_batch.forward_mode.is_extend():
			ret = self.forward_extend(forward_batch)

	def forward_decode(forward_batch: ForwardBatch):
		# For extend, we add input_embeds to kwargs
		return self.model.forward(
			forward_batch.input_ids,
			forward_batch.positions,
			forward_batch,
			**kwargs,
		)

```