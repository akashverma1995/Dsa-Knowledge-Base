# ml-dsa-notes

Personal ML, DSA, and MLOps reference notes — deployed as a documentation site via MkDocs Material.

🌐 **Live site:** https://Akashve1995.github.io/ml-dsa-notes

---

## Topics covered

- **DSA** — Graph algorithms (BFS, DFS, Dijkstra, Bellman-Ford, Floyd-Warshall, A*, Kruskal, Prim, Topo Sort, SCC, Max Flow, and more)
- **ML** — PEFT methods, NLP & Transformers, YOLO architectures, RAG pipelines *(coming soon)*
- **MLOps** — Docker, GCP, LLM fine-tuning *(coming soon)*
- **Edge AI** — MediaPipe, LiteRT / TFLite *(coming soon)*

---

## Run locally

```bash
# Clone the repo
git clone https://github.com/Akashve1995/ml-dsa-notes.git
cd ml-dsa-notes

# Install dependencies
pip install -r requirements.txt

# Serve locally at http://127.0.0.1:8000
mkdocs serve
```

## Deploy

Deployment is automatic. Every push to `main` triggers the GitHub Actions workflow which builds and pushes the site to the `gh-pages` branch.

To deploy manually:

```bash
mkdocs gh-deploy --force
```

---

## Stack

- [MkDocs](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- GitHub Actions + GitHub Pages
