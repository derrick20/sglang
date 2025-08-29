---
{"dg-publish":true,"permalink":"/work/sg-lang/schedule-batch/"}
---

Uses [[Molecules/Req\|Req]]

To prepare the encoder with the extend call:
- We know we want to place in the encoder cache slots.
- The decoder cache slot will just be a singular input for the start token
- There's more bookkeeping though:
	- If we've not cached it before (How about we assume that for now.)

```python title:managers/schedule_batch.py

class ScheduleBatch(ScheduleBatchDisaggregationDecodeMixin):
	reqs: List[Req]
	token_to_kv_pool_allocator
	req_to_token_pool
	tree_cache

	def prepare_encoder_info_extend(self, input_ids, seq_lens):
		# Do stuff by shifting from out_cache_loc to encoder_out_cache_loc?
		self.encoder_lens = (length of the inputs)
		We know that the 
	
	def prepare_for_extend(self):
		'''
		Allocate memory for req_to_token_pool (bs slots for the batch)
		Allocate any new pages overflowed into within token_to_kv_pool -> out_cache_loc
		Then, point the req_to_token_pool to these new out_cache_loc
			- Each request's extend  
		'''
		# Allocates memory
		bs = len(self.reqs)
		# Basically this, but it wraps with another method
		req_pool_indices = self.req_to_token_pool.alloc(bs)
		
		# Allocate for req
		if page_size == 1:
			out_cache_loc = self.alloc_token_slots(extend_num_tokens)
		else:
			# Complex paging logic, hmmph.
			out_cache_loc=  self.alloc_paged_token_slots()

		# Shift info into the encoder
		prepare_encoder_info_extend(, input_ids, seq_lens)


	def alloc_token_slots(self, num_tokens):
		# Free up space if needed
		if self.token_to_kv_pool_allocator.available_size() < num_tokens:
			self.tree_cache.evict(num_tokens)
		out_cache_loc = self.token_to_kv_pool_allocator.alloc(num_tokens) # This gives us the starting point 
		return out_cache_Loc
```