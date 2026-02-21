# Hospital MIP — MSE 433

This folder contains the MIP models, sensitivity analysis, and LaTeX formulation for the hospital flow project used in MSE 433.

Contents
- `1.ipynb` — primary notebook with model code and example runs.
- `hospital_mip.tex` — Overleaf-ready LaTeX file with the Original and Optimized MIP formulations.
- `sensitivity.py` — script that runs the Gurobi MIP sweeps (used to produce `sensitivity_results.csv`).
- `sensitivity_results.csv` — CSV output from the most recent sensitivity run.
- `generate_plots.py` — script to convert the CSV into PNG visualizations (produces `outputs/`).
- `outputs/` — contains generated PNGs and `summary.md` (plots created from `sensitivity_results.csv`).

Quick notes
- To regenerate sensitivity results locally you need a Gurobi license and Python packages used in `1.ipynb`.
- To regenerate plots:

```bash
python3 -m pip install -r requirements.txt
python3 generate_plots.py
```

How to produce the PDF of the LaTeX formulation

- Locally (requires TeX Live / pdflatex):

```bash
pdflatex -interaction=nonstopmode hospital_mip.tex
pdflatex -interaction=nonstopmode hospital_mip.tex
# or using latexmk
latexmk -pdf hospital_mip.tex
```

- On GitHub (automated): this repository includes a GitHub Actions workflow that compiles `hospital_mip.tex` to PDF on push/pull-request and uploads the compiled PDF as an artifact. See `.github/workflows/build-latex.yml`.

Notes about this environment
- I could not compile LaTeX here because `pdflatex` is not installed in this environment. The GitHub Actions workflow will compile the `.tex` file when you push to GitHub and produce a downloadable artifact.

If you want, I can also:
- Add more plots or annotated figures to `outputs/` before you push.
- Create a short PDF report combining plots and the rendered LaTeX (would need a TeX toolchain or CI to render math correctly).
