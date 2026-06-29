---
layout: default
title: OpenSearch-JVector
parent: Installing plugins
nav_order: 30

---

# OpenSearch-jvector plugin

The OpenSearch JVector Plugin enables running the nearest neighbor search on billions of documents across thousands of dimensions with the same ease as running any regular OpenSearch query. Aggregations and filter clauses can be applied to further refine similarity search operations. It's similarity search powers use cases such as product recommendations, fraud detection, image and video search, related document search, and more.

## Differences between `opensearch-jvector` and `k-NN`

The following table highlights the differences that matter most when choosing between the built-in k-NN plugin and `opensearch-jvector`:

| Aspect                   | `k-NN`                                                   | `opensearch-jvector`                                                      |
| :----------------------- | :----------------------------------------------------- | :---------------------------------------------------------------------- |
| **Vector engines**       | `Nmslib`, `Faiss`, `lucene`                            | `jvector` (primary), `lucene`                                           |
| **Concurrent ingestion** | Lucene index is not thread-safe for concurrent inserts | `jvector` supports concurrent inserts, enabling high-throughput ingestion |
| **Index update cost**    | Full rebuild required on merge                         | Incremental merges — no full rebuilds for updates                       |
| **Memory efficiency**    | In-memory indexing                                     | DiskANN-style quantization                                              |

## Usecases of plugin

`opensearch-jvector` is a strong choice when any of the following apply to your workload:

- **Your data grows continuously.** Incremental index updates and concurrent ingestion avoid the costly full rebuilds that occur with k-NN on high-update workloads.
- **Your dataset is larger.** DiskANN-style indexing with quantization-based reranking keeps recall high while memory usage stays bounded.
- **You care about search quality at high compression.** PQ with asymmetric distance computation (ADC) and SIMD acceleration delivers better recall than BQ at equivalent compression ratios.

Typical use cases include recommendation systems, image and video similarity search, semantic document search, and fraud detection pipelines that index millions to billions of vectors.

## Unique features

- **DiskANN Implementation (Pure Java)** - `jvector` provides a pure Java implementation of DiskANN-style approximate nearest neighbor (ANN) search optimized for memory-constrained environments. It eliminates the need for native libraries such as `Faiss` and avoids the complexity and overhead associated with JNI, simplifying deployment and maintenance.
- **Thread-Safe Design** - The index is fully thread-safe and supports concurrent updates and insertions with near-linear scalability as CPU cores increase. Unlike Lucene, which is not thread-safe at the index level, `jvector` enables high-throughput ingestion without relying on costly merge operations to achieve parallelism.
- **Quantized Index Construction** - `jvector` supports building indexes directly from quantized vectors, significantly reducing memory usage. This enables larger segment sizes, resulting in fewer segments overall and improved search performance.
- **Quantization Refinement During Merges** - The system refines quantization `codebooks` incrementally during merge operations. This approach improves search accuracy and recall without requiring a complete recomputation of `codebooks`, reducing computational overhead.
- **Incremental Index Updates** - `jvector` allows incremental insertion of vectors into existing persisted indexes. This eliminates the need for full index rebuilds, providing substantial efficiency gains for workloads involving frequent updates, particularly for large graph-based indexes.
- **Quantized DiskANN with Reranking** - `jvector` supports DiskANN-style quantization combined with reranking, delivering significant performance improvements for datasets larger than available memory. This approach is particularly effective for large-scale deployments where traditional in-memory indexing is not feasible.
- **Product Quantization (PQ) and Binary Quantization (BQ)** - `jvector` supports both PQ and BQ that Lucene offers. PQ is implemented with high-performance asymmetric distance computation (ADC), including SIMD optimizations and support for separate `codebooks`. PQ at higher compression ratios (e.g., 64×) provides better recall compared to BQ at lower compression levels (e.g., 32×).
- **Advanced Quantization Techniques** - `jvector` includes advanced capabilities such as Fused ADC, Non-Vector Quantization (NVQ), and Anisotropic PQ, enabling more efficient and accurate similarity computations beyond standard quantization approaches.
- **Cassandra Compatibility** - `jvector` is compatible with Cassandra, allowing seamless interoperability for vector data. This makes it easier to transfer and reuse encoded vector data between Cassandra and OpenSearch environments.

## Installation

**1. Remove Existing k-NN Plugin**

```bash
bin/opensearch-plugin remove opensearch-knn
```

**2. Install `opensearch-jvector` Plugin**

```bash
bin/opensearch-plugin install opensearch-jvector
```

## OpenSearch compatible features

- Product Quantization (Plugin version: 3.2.0.0)
- MMR Search (Plugin version: 3.6.0.0)
- Derived Sources (Plugin version: 3.6.0.0)

## Limitations

- `opensearch-jvector` plugin is not part of the default OpenSearch distribution
- `opensearch-knn` and `opensearch-jvector` cannot be installed simultaneously
- Hybrid search is not supported yet
