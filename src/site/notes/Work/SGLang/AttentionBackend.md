---
{"dg-publish":true,"permalink":"/work/sg-lang/attention-backend/"}
---


Calls the FlashAttention kernels

Notably, in :461
- It assigns the metadata's page_table to select
- In :597 - we may want to explore the metadata encoder_page_table
```python title:layers/attention/flashattention_backend.py

class FlashAttentionBackend(AttentionBackend):
	def init_forward_metadata(self, forward_batch):
	
	def forward_extend(self, q, k, v, layer: RadixAttention, forward_batch, save_kv_cache=True):
		'''
		We get the cache location allocated by the TokenToKVPoolAllocator for this specific batch.
		Then, we shift to use that buffer area within our batch-owned TokenToKVPool
		So, we're placing in the k,v we calculated upstream (during the Model's attention, projecting the hidden_states via the QKV weight)
		'''
		cache_loc = (forward_batch.out_cache_loc if not layer.is_cross_attention else forward_batch.encoder_out_cache_loc)
		# This captures the enc-dec version
		forward_batch.token_to_kv_pool.set_kv_buffer(
		layer, cache_loc, k, v, layer.k_scale, layer.v_scale)
	...
	'''
	Now, we can call attention on those collected cache.
	We just set the buffer, so now grab it back up.
	'''
	key_cache, value_cache = forward_batch.token_to_kv_pool.get_buffer(layer.layer_id)
	
	Reshape k/v with (-1, self.page_size, head_num, head_dim)

	# Complicated args
	result = flash_attn_with_kv(q, k, v, k_cache, v_cache, page_table, cache_seqlens)

def flash_attn_with_kv(q, k, v, k_cache, v_cache, page_table, ):
	'''
	Places the k and v values to the end of the cache at the needed slots?
	'''
```