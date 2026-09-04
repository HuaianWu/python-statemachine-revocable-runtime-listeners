FROM public.ecr.aws/x8v8d7g8/mars-base:latest
WORKDIR /app

# Git time-travel: clone, then make the repo's default branch point AT the base
# commit with no future history — a real branch checkout (not a detached HEAD),
# future commits/tags gc'd away so the reference solution can't leak from history.
ARG BASE_SHA=78e2b4c919faa2ac1c5196cb6ad5245fe4de7ee5
RUN git clone https://github.com/fgmacedo/python-statemachine . \
 && DEFAULT="$(git remote show origin | sed -n 's/.*HEAD branch: //p')" \
 && git checkout -B "$DEFAULT" "$BASE_SHA" \
 && git remote remove origin \
 && for b in $(git for-each-ref --format='%(refname:short)' refs/heads | grep -vx "$DEFAULT"); do git branch -D "$b" || true; done \
 && for t in $(git tag); do git merge-base --is-ancestor "$t" HEAD 2>/dev/null || git tag -d "$t"; done \
 && git reflog expire --expire=now --all \
 && git gc --prune=now \
 && { test ! -s .gitmodules || git submodule update --init --recursive; }
# Pinned from the repo's uv.lock (`uv export --frozen --no-default-groups --group <tests>`) at
# BASE_SHA, resolved for Linux / CPython 3.12 (the mars-base default interpreter), so the
# image matches upstream CI for this commit and does not drift when PyPI moves.
RUN pip install --no-cache-dir \
        "accessible-pygments==0.0.5" \
        "alabaster==1.0.0" \
        "anyio==4.14.0" \
        "asgiref==3.11.1" \
        "attrs==26.1.0" \
        "babel==2.18.0" \
        "beautifulsoup4==4.14.3" \
        "certifi==2026.2.25" \
        "cfgv==3.5.0" \
        "charset-normalizer==3.4.7" \
        "click==8.3.1" \
        "colorama==0.4.6" \
        "coverage==7.13.5" \
        "distlib==0.4.0" \
        "django==6.0.3" \
        "docutils==0.22.4" \
        "execnet==2.1.2" \
        "filelock==3.25.2" \
        "furo==2025.12.19" \
        "h11==0.16.0" \
        "identify==2.6.18" \
        "idna==3.18" \
        "imagesize==2.0.0" \
        "iniconfig==2.3.0" \
        "jinja2==3.1.6" \
        "jsonschema-specifications==2025.9.1" \
        "jsonschema==4.26.0" \
        "librt==0.8.1" \
        "markdown-it-py==4.0.0" \
        "markupsafe==3.0.3" \
        "mdit-py-plugins==0.5.0" \
        "mdurl==0.1.2" \
        "mypy-extensions==1.1.0" \
        "mypy==1.20.0" \
        "myst-parser==5.0.0" \
        "nodeenv==1.10.0" \
        "packaging==26.0" \
        "pathspec==1.0.4" \
        "pdbr==0.9.7" \
        "pillow==12.2.0" \
        "platformdirs==4.9.4" \
        "pluggy==1.6.0" \
        "pre-commit==4.6.0" \
        "py-cpuinfo==9.0.0" \
        "pydot==4.0.1" \
        "pygments==2.20.0" \
        "pyparsing==3.3.2" \
        "pyright==1.1.410" \
        "pytest-asyncio==1.4.0" \
        "pytest-benchmark==5.2.3" \
        "pytest-cov==7.1.0" \
        "pytest-django==4.12.0" \
        "pytest-mock==3.15.1" \
        "pytest-sugar==1.1.1" \
        "pytest-timeout==2.4.0" \
        "pytest-xdist==3.8.0" \
        "pytest==9.1.0" \
        "python-discovery==1.2.1" \
        "pyyaml==6.0.3" \
        "referencing==0.37.0" \
        "requests==2.34.2" \
        "rich==14.3.3" \
        "roman-numerals==4.1.0" \
        "rpds-py==2026.5.1" \
        "ruff==0.15.17" \
        "snowballstemmer==3.0.1" \
        "soupsieve==2.8.3" \
        "sphinx-autobuild==2025.8.25" \
        "sphinx-basic-ng==1.0.0b2" \
        "sphinx-copybutton==0.5.2" \
        "sphinx-gallery==0.21.0" \
        "sphinx==9.1.0" \
        "sphinxcontrib-applehelp==2.0.0" \
        "sphinxcontrib-devhelp==2.0.0" \
        "sphinxcontrib-htmlhelp==2.1.0" \
        "sphinxcontrib-jsmath==1.0.1" \
        "sphinxcontrib-mermaid==2.0.2" \
        "sphinxcontrib-qthelp==2.0.0" \
        "sphinxcontrib-serializinghtml==2.0.0" \
        "sqlparse==0.5.5" \
        "starlette==1.3.1" \
        "termcolor==3.3.0" \
        "typing-extensions==4.15.0" \
        "urllib3==2.7.0" \
        "uvicorn==0.42.0" \
        "virtualenv==21.2.0" \
        "watchfiles==1.1.1" \
        "websockets==16.0" \
    && pip install --no-cache-dir --no-deps -e . \
    && test -z "$(git status --porcelain)"

# v1.1 node-id scoring: pytest ships a native JUnit XML reporter (--junitxml),
# so no extra reporter dependency is required.

# Disable git commit hooks (pre-commit, husky etc.): dev-workflow tooling, not task content.
# Broken hook environments otherwise block the agent's (and oracle's) commits.
RUN cd /app && git config core.hooksPath /dev/null

CMD ["/bin/bash"]
