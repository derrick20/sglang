---
{"dg-publish":true,"permalink":"/work/sg-lang/mt-5-for-conditional-generation/"}
---


Built from GenerationMixin

prepare_decoder_input_ids_for_generation
- Ah yes, that generated the (batch_size, 1)
- See src/transformers/generation/utils.py/GenerationMixin

GenerationMixin
- We start with generate
	- Prepares model_kwargs with the cache
	- Then, the sample on :3595 has the generation loop
		- Prepares the model_inputs
		- This includes :551 prepare_inputs_for_generation()
			- This sets the past_key_values from the `**model_inputs`
			- 
	- THen passes this into the forward
		- 

Here's the structure:

T5Model
- Encoder T5Stack
	- Embedding if not done yet
		- E: (V, D)
		- We return E[input_ids] to select
			- Alternatively, you could say (B, N) becomes a one-hot encoding, (B, N, V).
			- Multiply by E
	- [T5Block] (12 layers)
		- T5SelfAttention
			- T5Attention
			- T5LayerNorm
		- T5LayerFF
- Decoder T5Stack
	- Embedding if not done yet
	- [T5Block]
		- T5SelfAttention
			- T5Attention
			- T5LayerNorm
		- T5CrossAttention
			- T5Attention
			- T5LayerNorm
		- T5LayerFF

LogitsProcessor
- LM Head (B, N, D) -> (B, N, V)
	- Multiply by $E^T$, since embedding is (V, D). 
		- This now flips the idea, where we have impure components, 
- From here, we slice out just the last `[:, -1, :] : (B, N, V) -> (B, V)`
- Now, we do the argmax to get the id, yielding (B, V) -> (B, 1)


The main idea is:
- We can understand the model and stack receive input_ids
- But each block will receive a hidden_states of the previous layer
	- Except for the first block, which technical receives input_ids