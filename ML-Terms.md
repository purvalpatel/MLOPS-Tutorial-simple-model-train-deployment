RAG
-----
RAG (Retrival Augmented Generation) <br>
R (Retrival)  - Fetch the information from your data source. (PDF, Websites, docs etc. ) <br>
A (Augmentation) - Insert (Augment) the retrived data into LLM's Prompt. <br>
G (Generation) - LLM uses the retrived context to generate accurate answer. <br>

employee-handbook.pdf <br>
You ask the LLM: <br>
```
“How many casual leaves per year do employees get?”
```
**Without RAG** → LLM hallucinates. <br>
**With RAG** → LLM pulls the answer from YOUR PDF. <br>
Provides answers from your data.

Neural Network:
---------
- Computer system inspred by humna brain.
- It recognize patterns and make decissions from data.

CNNs ( Convolutional Neural Netowkrs)
------------------------------
- Best for images, videos, Visual pattern recognition
- Automatically detect features like edges, shapes and textures.

Diffusion models:
--------------
- Image generation, Audio synthesis, denoising, creative generation tasks

Transformers ( Non LLM Context )
-----------------
- Sequential data like text, time series, image patches.
Use case :
- Time series forecasting ( Predict stock prices or sensor reading )
- Audio processing - speach recognition
- Protein folding

Weights:
-----
- learned values

Model Optimizations ( TensorRT Model Optimizer )
---------------------
1. Quantization -
 - Shrink the numbers
 - AI Model uses 32-bit floating numbers to store weights ( Their learned values )
 - Reduce that to 16,8,4-bit integrers
 - Make the model smaller in memory.

2. Distillation
   - Teach smaller students.
   - make smaller model almost same as big.
  
3. Sparsity
  - Remove unneccesary connections
  - turning of or removing connections that dont matter much.
  - Make model faster

4. Pruning
- Removing parts of neaural network that dont contribute much to the final result.

5. Speculative decoding:
- Guess now , check later.
