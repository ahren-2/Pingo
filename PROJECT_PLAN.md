# Pingo project plan

Status: planning; implementation has not started. The initial scope is based on the personal photo similarity search proposal. Estimates are part-time and should be revised after the hardware and dataset are chosen.

## Product goal

Build a local-first application that indexes a collection of images and finds visually similar images for an uploaded query without relying on filenames. Compare a pretrained image encoder with a version fine-tuned on a public labeled dataset using an NVIDIA GPU.

## Minimum viable product

- Select a local image folder, index supported images, and rebuild the index after files change. Handle empty folders and unreadable files without losing the rest of the collection.
- Upload a query image and display the top five available matches with similarity scores and useful error or empty states.
- Keep photos, query images, embeddings, metadata, and processing on the user's machine. Do not send photos to external services.
- Provide a way to compare the same query using the original and fine-tuned encoders. Prevent mixing an index with embeddings from another encoder or preprocessing configuration.
- Deliver training and evaluation scripts, an image indexer, an application, documented results, and a short demo.

## Proposed architecture

- Python and PyTorch for image preprocessing, encoder inference, training, and evaluation.
- FAISS for vector search, with normalized embeddings and an explicitly documented similarity metric. Start with an exact index for a small collection; optimize only after measuring a need.
- FastAPI for a local API, SQLite for image metadata and index bookkeeping, and React with TypeScript for the interface.
- Store only local paths and operational metadata in SQLite. Keep image files, model weights, datasets, indexes, credentials, and generated artifacts out of Git. Verify dependency and dataset licenses before choosing them.

These are starting choices, not claims that dependencies have been installed or tested. Confirm Windows and GPU compatibility before committing to a particular FAISS distribution or model.

## Milestones and acceptance criteria

### 1. Foundations and data (approximately week 1)

- Identify GPU model, available VRAM, CUDA/PyTorch compatibility, operating system, and development tools; record findings.
- Choose a small pretrained image encoder and a public labeled dataset with suitable licenses. Record dataset version, source, label semantics, sample counts, and any restrictions.
- Establish reproducible train, validation, and test splits before generating pairs. Ensure no identical or near-duplicate images cross splits; keep related captures together when metadata allows.
- Define disjoint held-out query and gallery images with at least one relevant gallery image per eligible query. Relevance means sharing the chosen label; document limitations of labels as a proxy for visual similarity.
- Set up the project structure, dependency management, basic checks, and a minimal local development workflow.

Done when a new developer can reproduce the dataset manifest and splits, and the GPU and dependencies have been verified on the target machine.

### 2. Baseline retrieval (approximately week 2)

- Implement consistent image decoding, resizing, normalization, batched embedding generation, and L2 normalization.
- Build a FAISS index and persist its encoder identity, preprocessing version, dimension, metric, and corresponding image IDs. Store image paths and indexing status in SQLite.
- Implement a command-line baseline that indexes a small collection and returns five ranked results with scores for a query.
- Create a reproducible held-out evaluation and save a baseline report with example successes and failures.

Done when rebuilding the same collection produces a usable index, known test queries retrieve the expected nearest vectors, and baseline metrics are reproducible.

### 3. Fine-tuning and comparison (approximately weeks 3-4)

- Generate positive and negative pairs or groups from training labels only. Choose and document a suitable contrastive or metric-learning objective and sampling strategy.
- Train on the available NVIDIA GPU with a small initial model and batch size; use validation data for configuration and checkpoint selection, never the test set.
- Save reproducible configuration, seeds, dataset and split identifiers, encoder version, training time, GPU/VRAM, and checkpoint provenance.
- Evaluate baseline and fine-tuned models on the identical held-out query/gallery split, preprocessing policy, relevance definition, and top-k settings. Record both results, even if fine-tuning does not improve them.

Done when a documented command reproduces the training setup and evaluation, and both models have a comparable report with representative successes and failures.

### 4. Local application (approximately week 5)

- Add local API endpoints for indexing/rebuilding, index status, and image search, with validation and clear error responses.
- Build the React interface for selecting a collection, starting a rebuild, uploading a query, viewing five scored results, and switching encoder versions.
- Handle missing or changed images, corrupt inputs, no index, and fewer than five results. Keep the UI responsive during indexing and communicate progress or status.
- Bind the API to localhost by default, validate paths and file types, avoid unrestricted local-file serving, and avoid logging photo contents or unnecessary sensitive paths.

Done when a user can complete the full local indexing and search flow for both encoders, including a rebuild after changes, without a cloud service.

### 5. Validation and delivery (approximately week 6)

- Test indexing, model/index compatibility, similarity ordering, evaluation calculations, API validation, and the primary UI flow. Include corrupt, empty, duplicate, and changed-file cases.
- Document setup, privacy behavior, model and dataset licenses, limitations, reproducible commands, evaluation findings, and troubleshooting in the GitHub README or linked documentation.
- Record a short demo: search with a sample image, compare both encoders, rebuild the index, and discuss a failure and measured results.

Done when the project can be set up from a clean checkout, the documented end-to-end flow works, and the evaluation report and demo are available.

## Evaluation protocol

For each eligible held-out query, retrieve up to five gallery images, excluding the query itself and any prohibited duplicate. Define recall@5 as the number of relevant gallery images in the top five divided by the total number of relevant gallery images for that query; report the mean across queries. Also report hit rate@5 (the fraction of queries with at least one relevant result in the top five) to make the practical search result easy to interpret. State the query and gallery sizes, number of excluded queries, label rule, split policy, and whether results are exact search. Use the same fixed test set and code for both models; do not tune against the test results.

Record indexing time, search latency, hardware, and model size as supporting measurements. Do not invent a quality threshold before establishing the baseline; the minimum success criterion is a transparent, repeatable comparison and a usable local search flow.

## Deferred work

Natural-language search, face recognition, cloud sync, mobile clients, approximate search at large scale, and user-feedback-driven retraining are outside the initial scope. A later feedback experiment should reserve a fresh test set that is not used to form training pairs.

## Next decisions

1. Confirm the NVIDIA GPU model and available VRAM.
2. Select the dataset and pretrained encoder after checking their licenses and hardware requirements.
3. Agree on supported image formats and an initial collection size.
4. Decide whether the long-term repository should remain private or become public.
