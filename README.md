# NetRadar documentation

Source of <https://netradar-testbed.github.io>, built with
[MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and deployed by
GitHub Actions on every push to `main`.

Preview locally:

```bash
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt
.venv/bin/mkdocs serve        # http://127.0.0.1:8000
```

Pages live in `docs/`; the navigation is in `mkdocs.yml`. Changes go through
pull requests (see CONTRIBUTING in the organisation profile).

Content is licensed under Apache-2.0, like the NetRadar code.
