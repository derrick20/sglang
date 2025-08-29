---
{"dg-publish":true,"permalink":"/work/sg-lang/token-to-kv-pool-allocator/"}
---


Broad concept:
- We have a free_group
	- Python list
	- start_free_group causes all new free's to be placed into this list
	- At end_free_group, we transfer to concatenate within the release pages
- Release pages
	- Torch tensor
	- `free` concatenates freed pages into these `release_pages`


This free_group is used within [[SchedulerOutputProcessorMixin\|SchedulerOutputProcessorMixin]]
:218, sets it.
In :227-239, it frees up a specific page.
Since there are `batch_size` requests, we need to have each of those be freed. We'd like to avoid too many concatenating/sorting calls I assume.


This references allocation kernels too:
- At allocator.py:367
	- We have the alloc_decode_kernel
	- It's pretty fun, similar to [[segment tree\|segment tree]] or [[sqrt decomposition\|sqrt decomposition]] logic
- If we think about the extend logic.
- 

```python title:mem_cache/allocator.py

class BaseTokenToKVPoolAllocator:
    def free_group_begin(self):
        self.is_not_in_free_group = False
        self.free_group = []

    def free_group_end(self):
        self.is_not_in_free_group = True
        if self.free_group:
            self.free(torch.cat(self.free_group))

	def merge_and_sort_free(self):
		'''
		# Absorbs in the released pages, sorts to organize contiguously, then clear released pages
		'''
        if len(self.release_pages) > 0:
            self.free_pages = torch.cat((self.free_pages, self.release_pages))
            self.free_pages, _ = torch.sort(self.free_pages)
            self.release_pages = torch.empty(
                (0,), dtype=self.release_pages.dtype, device=self.device
            )
            
class TokenToKVPoolAllocator(BaseTokenToKVPoolAllocator):
	def alloc(self, need_size):
		'''
		# If we need more memory, free and merge
		# If still not enough, return None
		# slice off the amount needed
		'''
		if need_size > len(self.free_pages):
			self.merge_and_sort_free()
		if need_size > len(self.free_pages):
			return None
		select_index = self.free_pages[:need_size]
		self.free_pages = self.free_pages[need_size:]
		return select_index

	def free(self, free_index):
		'''
		# If in free_group (rare, so we track with the negated bool), add to that list
		# Otherwise, concatenate to release pages
		'''
		if free_index.numel() == 0:
			return
		if self.is_not_in_free_group:
			self.release_pages = torch.cat((self.release_pages, free_index))
		else:
			self.free_group.append(free_index)


class PagedTokenToKVPoolAllocator(BaseTokenToKVPoolAllocator):
	def alloc(self, need_size):
		num_pages = n
```

