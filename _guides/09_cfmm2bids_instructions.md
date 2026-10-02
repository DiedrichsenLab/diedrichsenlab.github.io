---
layout: page
title: CFMM2BIDS Instructions
date: 2026-10-02
category: fMRI
description: How to use cfmm2bids to BIDS your CFMM (f)MRI data.
---
## 0. Prerequisites

Install pixi (the package manager used to run the workflow), then close and reopen the terminal so the `pixi` command works:

```bash
curl -fsSL https://pixi.sh/install.sh | sh
```

Create a UWO credentials file so the workflow can log in to the CFMM DICOM server:

```bash
nano ~/.uwo_credentials
```

Put your UWO username (the part before @uwo.ca in your UWO email) on the first line and your UWO password on the second line, with nothing else in the file. For example:

```
jsmith2
YourPassword123
```

Save with Ctrl+O, Enter, then exit with Ctrl+X. Make the file readable only by you, and never commit it to GitHub:

```bash
chmod 600 ~/.uwo_credentials
```

In your config file, make sure `credentials_file` points to it: `credentials_file: ~/.uwo_credentials`.

Check that Apptainer is installed (needed for gradcorrect).

```bash
apptainer --version
```


## 1. Clone cfmm2bids

```bash
git clone https://github.com/akhanf/cfmm2bids
cd cfmm2bids
pixi install
```

This will give folder “cfmm2bids”, which contains folders: “config”, “heuristics”, “resources”, “workflow”. Run all commands in this guide from the “cfmm2bids” folder (not from inside “config”).


## 2. Create experiment-specific config file

Create experiment-specific config file to identify subjects for extraction, by using config.yml as template:

1. To create a new file to work on:

    ```bash
    cp config/config.yml config/config_[experiment_name].yml
    ```

2. Edit section `pattern: '.*_(S[0-9]+)_[0-9]+$'` in your new config file so it extracts the subject number. This pattern is based on how the Patient’s Name appears in CFMM data browser (ex: `2026_06_23_S04_2` can be represented by `.*_(S[0-9]+)_[0-9]+$`, which captures `S04` as the subject).


3. To choose which subjects to extract, add them under `study_filter_specs` (keep the quotes):

```yaml
    study_filter_specs:
      include:
        - "subject == 'S04'"
```

4. Set your study name under `search_specs`:

```yaml
    search_specs:
      - dicom_query:
          study_description: PI^StudyName^*
```

5. To enable gradcorrect, set `enable: true` beneath “6. GRADCORRECT STAGE” in config file, and set the coefficient file path:

```yaml
    grad_coeff_file: /srv/software/gradcorrect/coeff_AC84.grad
```

## 3. Adjust heuristics file

Adjust heuristics file “cfmm_base.py” as needed in “cfmm2bids/heuristics” for the experiment. Check the series names for your scans in the CFMM data browser.

This could require creating new “keys” (Ex: “func_bold”, “func_sbref”) that follow the data naming in the experiment, for example:

```python
func_bold = create_key('{bids_subject_session_dir}/func/{bids_subject_session_prefix}_task-language_run-{item:02d}_bold')
```

for naming `_task-language_run-01_bold`

Each key also needs a rule in `infotodict` that matches the series description from `dicominfo.tsv`, for example:

```python
if 'bold_language_AP' in s.series_description and s.dim4 > 100:
    info[func_bold].append({'item': s.series_id})
```

## 4. Run one subject, one session at a time

```bash
pixi run snakemake -C head=1 --configfile config/config_[experiment_name].yml --use-apptainer --apptainer-args "--bind /srv" --cores all
```

`head=1` processes the first subject only. Change 1 to N to process the first N subjects, or remove `-C head=1` to process all subjects.

This will compute steps “query”, “filter”, “download”, “convert”, “fix”, and “gradcorrect” (if enabled), and create files in the “results” folder of cfmm2bids (in folders `0_query`, `1_filter`, `2_download`, `3_convert`, `4_fix`, `5_gradcorr`). Extracted scans (uncorrected) will be in:

```
results/4_fix/bids/sub-S0X/ses-X/
```

The final dataset (gradient-corrected if gradcorrect is enabled) will be in `bids/` in the cfmm2bids folder.
    
## 6. Run each step of the conversion individually

To run each step of the conversion individually, do the following in order:

1. **Download step:**

```bash
    pixi run snakemake download -C head=1 --configfile config/config_[experiment_name].yml --cores all
```

2. **Convert step:**

```bash
    pixi run snakemake convert -C head=1 --configfile config/config_[experiment_name].yml --cores all
```

3. **Fix step:**

```bash
    pixi run snakemake fix -C head=1 --configfile config/config_[experiment_name].yml --cores all
```

4. **Gradcorrect step:** (this runs the remaining steps, including gradcorrect, and puts the corrected scans in `results/5_gradcorr/` and the final dataset in `bids/`)

```bash
    pixi run snakemake -C head=1 --configfile config/config_[experiment_name].yml --use-apptainer --apptainer-args "--bind /srv" --cores all
```

5. **To check the output** (especially in case of failures), run the BIDS validator on the final dataset:

```bash
    pixi run bids-validator-deno bids --format text --ignoreWarnings
```