# Quickstart: Build and Inspect the NDNSF-Repo Slides

## Build

```bash
cd /home/tianxing/NDN/ndn-service-framework/docs/NDNSF-REPO/slides
pdflatex -interaction=nonstopmode -halt-on-error main.tex
pdflatex -interaction=nonstopmode -halt-on-error main.tex
pdfinfo main.pdf
```

Expected result: `main.pdf` exists, reports a 16:9 page size, and has no more than 20 pages.

## Text and Layout Checks

```bash
rg -n "Overfull|Underfull|LaTeX Error" main.log
pdftotext -layout main.pdf /tmp/ndnsf-repo-slides.txt
pdftoppm -png -r 120 main.pdf /tmp/ndnsf-repo-slide
```

Inspect the rendered pages as a contact sheet and open selected dense pages at full resolution. Verify:

- no clipping or overlap;
- readable labels and tables;
- one main idea per frame;
- correct current/total page numbers;
- no unsupported claim that repo nodes decrypt or validate opaque app payloads;
- no application-specific policy moved into the repo.

## Evidence References

- `NDNSF-DistributedRepo/README.md`
- `NDNSF-DistributedRepo/include/ndnsf-distributed-repo/RepoClient.hpp`
- `NDNSF-DistributedRepo/include/ndnsf-distributed-repo/RepoNode.hpp`
- `NDNSF-DistributedRepo/include/ndnsf-distributed-repo/RepoCore.hpp`
- `NDNSF-DistributedRepo/include/ndnsf-distributed-repo/RepoTypes.hpp`
- `NDNSF-DistributedRepo/src/RepoTypes.cpp`
- `examples/python/NDNSF-DistributedRepo/generic_object_store/README.md`
- `Experiments/NDNSF_DistributedRepo_Generic_Minindn.py`
