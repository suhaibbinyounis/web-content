---
title: Learning Transformers by Building One
date: 2025-06-17T08:31:52.780Z
description: Dive deep into the Transformer architecture by understanding its fundamental components and how to conceptually build one from scratch. Unravel the magic behind modern AI.
tags:
    - AI
    - LLM
    - Transformers
    - Deep Learning
    - Machine Learning
    - NLP
    - PyTorch
categories:
    - AI
    - Deep Learning
    - Tutorials
    - NLP
comments: true
---

The world of Artificial Intelligence has been irrevocably changed by the Transformer architecture. From powering the most advanced Large Language Models (LLMs) like GPT-4, Llama, and Gemini, to revolutionizing image recognition and even scientific discovery, its impact is undeniable. While it's easy to use pre-built Transformer models, truly understanding their power comes from dissecting and, conceptually at least, building one yourself.

This post will guide you through the core components of the Transformer, explaining the "why" behind each design choice, giving you a solid foundation to build your own, or at least appreciate the engineering marvel it truly is.

## The Genesis: "Attention Is All You Need"

Before Transformers, recurrent neural networks (RNNs) and convolutional neural networks (CNNs) were the workhorses of sequence modeling. RNNs, despite their ability to handle sequences, struggled with long-range dependencies and were inherently sequential, making parallelization difficult. Then, in 2017, a team of researchers from Google published a groundbreaking paper: ["Attention Is All You Need"](https://arxiv.org/abs/1706.03762).

This paper introduced the Transformer architecture, which completely eschewed recurrence and convolutions in favor of a novel mechanism called "multi-head self-attention." This simple yet powerful idea unlocked unprecedented parallelization and dramatically improved the model's ability to capture relationships across long distances in sequences.

## The Encoder-Decoder Foundation

The original Transformer, designed for sequence-to-sequence tasks like machine translation, features an encoder-decoder architecture.

*   **Encoder**: Processes the input sequence (e.g., a sentence in English) and transforms it into a rich, contextual representation.
*   **Decoder**: Takes the encoder's output and generates the output sequence (e.g., the translated sentence in French) one token at a time, using the context from both the encoder and the previously generated tokens.

While many modern LLMs primarily use an encoder-only (like BERT) or decoder-only (like GPT) architecture, understanding the full encoder-decoder model provides the most comprehensive view. We'll break down the key building blocks that make up both.

## Building Blocks of a Transformer

To conceptually build a Transformer, let's explore its essential components in detail.

### 1. Input Embeddings & Positional Encoding

Neural networks don't understand words or characters directly; they understand numbers. The first step for any NLP model is to convert discrete input tokens into continuous vector representations called **embeddings**.

*   **Input Embeddings**: Each unique word or sub-word (token) in your vocabulary is mapped to a dense vector. These vectors are learned during training and capture semantic meanings. For instance, "king" and "queen" might have similar vectors, reflecting their semantic relationship.
    *   *Implementation thought*: In PyTorch, this is typically `torch.nn.Embedding`. For a vocabulary of size `V` and an embedding dimension `d_model`, this would be `nn.Embedding(V, d_model)`.

The Transformer processes entire sequences in parallel, unlike RNNs which process token by token. This parallel processing, while efficient, means the model loses information about the *order* of tokens in the sequence. "Dog bites man" and "Man bites dog" would produce the same set of embeddings if order isn't preserved.

*   **Positional Encoding**: To re-inject sequence order information, the Transformer adds "positional encodings" to the input embeddings. These are unique vectors for each position in the sequence, allowing the model to know if a token is at the beginning, middle, or end. The original paper used fixed sinusoidal functions for these encodings:

    $PE_{(pos, 2i)} = \sin(pos / 10000^{2i/d_{model}})$
    $PE_{(pos, 2i+1)} = \cos(pos / 10000^{2i/d_{model}})$

    where `pos` is the position, `i` is the dimension, and `d_model` is the embedding dimension. This choice allows the model to potentially learn relative positions easily.
    *   *Why sinusoidal?*: It allows generalization to sequence lengths longer than those seen during training, as the relative positions maintain a consistent relationship.
    *   *Implementation thought*: This involves creating a matrix of these sinusoidal values and adding them element-wise to the word embeddings. This typically happens as part of an `InputEmbedding` or `EmbeddingWithPositionalEncoding` module.

The combined input to the first layer of the Transformer is `Input Embedding + Positional Encoding`.

### 2. Multi-Head Attention

This is the beating heart of the Transformer. Attention allows the model to weigh the importance of different parts of the input sequence when processing a specific token.

At its core, attention can be thought of as retrieving information. For each token, we ask: "What other tokens in the sequence are relevant to understanding *this* token?"

#### Scaled Dot-Product Attention

The fundamental attention mechanism is the Scaled Dot-Product Attention. It takes three inputs: Query (Q), Key (K), and Value (V).
*   **Query (Q)**: Represents the current token we're trying to understand.
*   **Key (K)**: Represents all other tokens in the sequence that we might need to "look up."
*   **Value (V)**: The actual information (representations) associated with those other tokens.

The process is:
1.  **Similarity Score**: Compute the dot product between Q and K for all tokens. This measures how "similar" (or relevant) a query is to each key.
2.  **Scaling**: Divide the scores by the square root of the dimension of the keys ($\sqrt{d_k}$). This scaling is crucial to prevent the dot products from becoming too large, which can push the softmax function into regions with very small gradients, leading to vanishing gradients.
3.  **Softmax**: Apply a softmax function to the scaled scores. This turns the scores into a probability distribution, indicating the "attention weights" – how much each token's value contributes to the output.
4.  **Weighted Sum**: Multiply the attention weights by the Values (V) and sum them up. This produces the final attention output for the query token, which is a weighted sum of all value vectors.

The formula is:
$\text{Attention}(Q, K, V) = \text{softmax}(\frac{QK^T}{\sqrt{d_k}})V$

*   *Implementation thought*: This involves `torch.matmul`, `softmax`, and division. Q, K, V are typically derived from the input by three separate linear transformations.

#### Multi-Head Mechanism

Instead of performing attention once, Multi-Head Attention performs it multiple times in parallel, using different, learned linear projections of Q, K, and V. Each "head" learns to focus on different aspects or relationships within the sequence.

1.  **Projections**: Q, K, V are projected linearly `h` times (where `h` is the number of heads) into different sub-spaces, usually with a smaller dimension ($d_k = d_v = d_{model}/h$).
2.  **Parallel Attention**: Scaled Dot-Product Attention is computed independently for each of these `h` projected Q, K, V sets.
3.  **Concatenation**: The outputs from all `h` attention heads are concatenated.
4.  **Final Linear Projection**: The concatenated output is then linearly projected back to the original `d_model` dimension.

*   *Why Multi-Head?*: It allows the model to jointly attend to information from different representation subspaces at different positions. For instance, one head might focus on syntactic relationships, while another focuses on semantic ones.
*   *Implementation thought*: This involves multiple `nn.Linear` layers to create Q, K, V for each head, then reshaping, applying attention, concatenating, and a final `nn.Linear` layer.

#### Masking (Crucial for Decoder and Padding)

Two types of masks are used with attention:

1.  **Padding Mask**: Input sequences often have varying lengths and are padded with special "padding tokens" to make them uniform for batch processing. This mask ensures that the model doesn't attend to these meaningless padding tokens. It typically sets the attention scores for padding tokens to negative infinity *before* the softmax, making their weights effectively zero.
2.  **Look-Ahead (Causal) Mask**: Used exclusively in the decoder's *self-attention* layer. When predicting the next token in an output sequence, the decoder should only be able to attend to tokens that have *already been generated* (or are to its left in the sequence). This mask prevents it from "cheating" by looking at future tokens. It's a triangular mask that sets attention scores for future positions to negative infinity.

*   *Implementation thought*: Creating a boolean mask and applying it before the softmax step: `scores.masked_fill(mask == 0, -1e9)`.

### 3. Feed-Forward Network (FFN)

After the attention sub-layer, each position in the sequence passes independently through a simple, fully connected feed-forward network. This is a two-layer neural network with a ReLU activation in between:

$FFN(x) = \max(0, xW_1 + b_1)W_2 + b_2$

*   *Implementation thought*: Typically `nn.Linear(d_model, d_ff)`, `nn.ReLU()`, then `nn.Linear(d_ff, d_model)`. Here, `d_ff` is often 4 times `d_model` (e.g., 2048 or 4096).
*   *Why FFN?*: This layer allows the model to perform non-linear transformations on the attention output, enriching its representation. It acts independently on each position, further contributing to parallelizability.

### 4. Add & Normalize (Residual Connections & Layer Normalization)

Both the Multi-Head Attention and the Feed-Forward Network sub-layers in the Transformer are wrapped with two crucial components:

*   **Residual Connections (Skip Connections)**: Inspired by ResNet, these add the input of a sub-layer to its output. If the input to a sub-layer is `X` and its function is `Sublayer(X)`, the output is `X + Sublayer(X)`.
    *   *Why?*: They help mitigate the vanishing gradient problem, allowing deeper networks to be trained effectively. They ensure that information can flow directly through the network, preventing earlier layers from being forgotten.
    *   *Implementation thought*: Simply `input_tensor + sublayer_output`.

*   **Layer Normalization**: Applied after the residual connection, Layer Normalization normalizes the sum of the input and the sub-layer output across the *feature dimension* for each individual sample in the batch. Unlike Batch Normalization which normalizes across the batch dimension, Layer Norm is independent of batch size.
    *   *Why?*: It stabilizes training and speeds up convergence by maintaining the mean and variance of activations.
    *   *Implementation thought*: `nn.LayerNorm(d_model)`.

The order is crucial: `Add&Norm(x) = LayerNorm(x + Sublayer(x))`. The original paper used `Add&Norm` after each sub-layer.

## Assembling the Blocks: Encoder and Decoder

### The Encoder Block

A single Encoder block consists of:
1.  **Multi-Head Self-Attention** sub-layer (with `Add & Norm`). This allows tokens within the input sequence to attend to each other.
2.  **Feed-Forward Network** sub-layer (with `Add & Norm`).

Multiple identical Encoder blocks are stacked on top of each other (e.g., 6 blocks in the original paper). The output of one block becomes the input to the next.

### The Decoder Block

A single Decoder block is more complex and consists of three sub-layers:
1.  **Masked Multi-Head Self-Attention** sub-layer (with `Add & Norm`). This is similar to the encoder's self-attention, but with the crucial **look-ahead mask** to prevent attending to future tokens. It allows the decoder to understand the context of the output tokens generated so far.
2.  **Multi-Head Encoder-Decoder Attention** sub-layer (with `Add & Norm`). Here, the Queries come from the *output of the previous masked self-attention layer* (i.e., the partially generated target sequence), while the Keys and Values come from the *output of the Encoder stack*. This layer allows the decoder to focus on relevant parts of the input source sequence when generating each output token.
3.  **Feed-Forward Network** sub-layer (with `Add & Norm`).

Similar to the encoder, multiple Decoder blocks are stacked.

### Output Layer

The final output of the top Decoder block is passed through a linear layer, followed by a softmax activation function. This transforms the high-dimensional representation into a probability distribution over the entire vocabulary, indicating the likelihood of each possible next token.

*   *Implementation thought*: `nn.Linear(d_model, vocab_size)`, then `torch.nn.functional.log_softmax` (for log probabilities, common for training with NLLLoss) or `softmax`.

## Putting It All Together: A Conceptual Code Structure

If you were building this in PyTorch, your `Transformer` class might look something like this:

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math

class PositionalEncoding(nn.Module):
    # ... (implementation for sinusoidal PE)

class MultiHeadAttention(nn.Module):
    # ... (implementation for Q, K, V projections, scaled dot-product, masking, final projection)

class PositionwiseFeedForward(nn.Module):
    # ... (implementation for two linear layers with ReLU)

class EncoderLayer(nn.Module):
    # ... (self-attention, FFN, two Add & Norm blocks)

class DecoderLayer(nn.Module):
    # ... (masked self-attention, encoder-decoder attention, FFN, three Add & Norm blocks)

class Encoder(nn.Module):
    # ... (stack of EncoderLayers)

class Decoder(nn.Module):
    # ... (stack of DecoderLayers)

class Transformer(nn.Module):
    def __init__(self, vocab_size, d_model, nhead, num_encoder_layers,
                 num_decoder_layers, dim_feedforward, max_seq_len, dropout=0.1):
        super().__init__()
        self.src_embed = nn.Embedding(vocab_size, d_model)
        self.tgt_embed = nn.Embedding(vocab_size, d_model)
        self.pos_encoder = PositionalEncoding(d_model, dropout, max_seq_len)

        self.encoder = Encoder(EncoderLayer(d_model, nhead, dim_feedforward, dropout), num_encoder_layers)
        self.decoder = Decoder(DecoderLayer(d_model, nhead, dim_feedforward, dropout), num_decoder_layers)

        self.output_linear = nn.Linear(d_model, vocab_size)

    def forward(self, src, tgt, src_mask, tgt_mask, src_padding_mask, tgt_padding_mask, memory_padding_mask):
        src_embedded = self.pos_encoder(self.src_embed(src))
        tgt_embedded = self.pos_encoder(self.tgt_embed(tgt))

        memory = self.encoder(src_embedded, src_padding_mask)
        output = self.decoder(tgt_embedded, memory, tgt_mask, tgt_padding_mask, memory_padding_mask)
        
        return F.log_softmax(self.output_linear(output), dim=-1)

# Note: This is a simplified structural outline. Each module would contain its own parameters,
# layer normalization, residual connections, and specific forward pass logic.
# Dropout layers are also typically added after attention and FFN.
```

## Why Transformers Are So Effective

Understanding the components helps us appreciate why Transformers became so dominant:

1.  **Parallelization**: Unlike RNNs, which process sequences sequentially, the attention mechanism allows Transformers to process all tokens in a sequence simultaneously. This makes training significantly faster on modern hardware (GPUs, TPUs).
2.  **Long-Range Dependencies**: The attention mechanism directly models relationships between any two positions in a sequence, regardless of their distance. This contrasts with RNNs, where information has to propagate through many time steps, often leading to vanishing or exploding gradients for very long dependencies.
3.  **Scalability**: The modular and parallelizable nature of Transformers allows for the creation of extremely deep and wide models. As computational resources and datasets grew, Transformers scaled exceptionally well, leading to the "emergent abilities" observed in LLMs.
4.  **Transfer Learning**: Pre-training large Transformer models on vast amounts of unlabelled text data (e.g., using masked language modeling or next-token prediction) and then fine-tuning them on specific downstream tasks has proven incredibly effective. This pre-train/fine-tune paradigm unlocked state-of-the-art performance across diverse NLP tasks.

## Challenges and Considerations

While revolutionary, Transformers are not without their limitations:

1.  **Quadratic Complexity**: The standard self-attention mechanism has a computational complexity that is quadratic with respect to the sequence length ($O(L^2)$ where $L$ is sequence length). This means processing very long sequences (e.g., entire books, high-resolution images) can become prohibitively expensive in terms of memory and computation.
2.  **Memory Footprint**: Storing the Q, K, V matrices and attention weights also incurs significant memory costs, scaling quadratically with sequence length.
3.  **Data Hungry**: Training large Transformer models from scratch requires enormous amounts of data to achieve their full potential.
4.  **Interpretability**: While attention weights can offer some insights into what the model is focusing on, fully interpreting the internal workings of massive Transformer models remains a challenge.

Researchers are actively working on addressing the quadratic complexity with more efficient attention mechanisms (e.g., [Linformer](https://arxiv.org/abs/2006.04768), [Performer](https://arxiv.org/abs/2009.14794), [FlashAttention](https://arxiv.org/abs/2205.14135)) and exploring alternatives to attention for long sequences.

## Conclusion

Building a Transformer, even conceptually, is one of the best ways to truly grasp the principles that underpin the AI revolution. From the ingenious positional encodings that re-inject sequence order to the powerful multi-head attention mechanism that captures intricate dependencies, each component plays a vital role.

While the original "Attention Is All You Need" paper laid the foundation, the Transformer has evolved into countless variants and applications. By understanding its core, you're not just learning a specific architecture; you're gaining insight into the fundamental building blocks of modern artificial intelligence. Now, go forth and build something!