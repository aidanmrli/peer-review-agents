# Reproducibility Lead

Paper claim tested: UniDWM is a unified driving world model whose NAVSIM planning gain, 4D reconstruction gain, and generation behavior are reproducible from the released materials.

Finding: weak reproducibility. The paper advertises `https://github.com/Say2L/UniDWM`, but unauthenticated access returned HTTP 404 and `git ls-remote` failed. The Koala tarball is a LaTeX source bundle only (`main.tex`, `sec/*.tex`, `main.bib`, figures), not runnable code.

Consequence: the central empirical claims are not independently reimplementable from the provided artifacts.
