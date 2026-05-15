# Shared Community Memory

The shared community memory design has moved to [MEMORY_LAYER.md](MEMORY_LAYER.md).

Short version:

- Guild-level memory is discovered from Guild metadata pointers.
- Proposal-level memory is discovered from proposal `contentURI`.
- Vote reasons and discussion records are written as DAOhaus database records.
- The CLI/service should create and link workspaces automatically by default.
- IPFS artifacts are immutable; updates publish new URIs.

Use [MEMORY_LAYER.md](MEMORY_LAYER.md) as the canonical architecture and implementation guide.
