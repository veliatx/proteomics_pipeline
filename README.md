# Proteomics Pipeline Documentation: proteomics_pipeline
## Documentation Generated by Ian Smith

## Overview
This codebase implements a proteomics data analysis pipeline with support for both Label-Free Quantification (LFQ) and Tandem Mass Tag (TMT) MS2/MS3 workflows. The pipeline handles raw mass spectrometry data processing, database searching, and quantification.

## Dependencies

### Python Package Dependencies
<ul>
    <li>sys</li>
    <li>pathlib</li>
    <li>logging</li>
    <li>datetime</li>
    <li>typing</li>
    <li>functools</li>
    <li>pandas</li>
    <li>numpy</li>
    <li>tqdm</li>
    <li>click</li>
    <li>ppx</li>
    <li>shutil</li>
    <li>mokapot</li>
    <li>importlib</li>
    <li>pyteomics</li>
    <li>statistics</li>
</ul>


### External Software Dependencies
<ul>
    <li>Comet Search Engine - will require path</li>
    <li>MSConvert (ProteoWizard) - will require path</li>
</ul>

## Key Components

### Main Pipeline Functions
<ul>
    <li>Raw file extraction from PRIDE or internal repositories</li>
    <li>Conversion of instrument vendor raw files to mzML format</li>
    <li>Database searching using Comet with some standard parameter defaults and custom params options</li>
    <li>MS1 feature detection using Biosaur2 (for LFQ)</li>
    <li>TMT reporter ion quantification (for TMT) for MS2/MS3</li>
    <li>FDR control using Mokapot with a joint model and result processing and quantification append.</li>
</ul>

## Example Usage

The pipeline can be run either through command line interface or Python scripts. Recommend running in Jupyter Notebook. Example jupyter notebook accessed in package. This pipeline enables AWS (s3) and dockerized msconvert functionality (please use help command to find arguments in cli functions to enable AWS-based functionality).Below is an example workflow using Python:

### 1a. List Files from PRIDE Repository
```python
from proteomics_pipeline.main import pride_repo_file_display

file_list = pride_repo_file_display(
    path = "C:/project_path_name/",
    pride_repo = "PXD038358",
    file_extension = "*.raw"
)
```

### 1b. List Files from Internal Repository
```python
from proteomics_pipeline.main import internal_repo_file_display

local_files = internal_repo_file_display(
    path = "C:/project_path_name/",  
    interal_repo_path = "C:/path_repo_location_raw_files/",
    file_extension = "*.raw"
)
```

### 2. Extract Raw Files
You can extract files from either PRIDE or internal repository:

#### From PRIDE:
```python
from proteomics_pipeline.main import raw_file_extract

raw_file_extract(
    path = "C:/project_path_name/",
    pride_or_internal = "pride",
    pride_repo = "PXD038358",
    file_list = ['APEX-GABARAPL2_1.raw', 
                 'APEX-GABARAPL2_2.raw',
                 'APEX-GABARAPL2_3.raw']
)
```

#### From Internal Repository:
```python
raw_file_extract(
    path = "C:/project_path_name/",
    pride_or_internal = "internal",
    interal_repo_path = "C:/path_repo_location_raw_files/",
    file_extension = "*.raw"
)
```

### 3. Convert RAW to mzML
```python
from proteomics_pipeline.main import raw_to_mzml

raw_to_mzml(
    path = "C:/project_path_name/",
    path_msconvert = "C:/Users/local_path_to_msconvert/ProteoWizard/msconvert.exe"
)
```

### 4a. Database Search with default params
```python
from proteomics_pipeline.main import db_search

db_search(
    path = "C:  /project_path_name/",
    path_comet = "C:/path_to_comet_exe/Comet/comet.win64.exe",
    param_default = "human_tmt11_tryptic_itms2",
    species = "human"
)
```

### 4b. Database Search with custom params
```python
from proteomics_pipeline.main import db_search

db_search(
    path = "C:/project_path_name/",
    path_comet = "C:/path_to_comet_exe/Comet/comet.win64.exe",
    params = "C:/path_to_custom_params"
    ## ensure custom params and corresponding fasta are in path of path argument and are the only onces present in the folder
)
    ```

### 5a. For LFQ: MS1 Feature Detection
```python
from proteomics_pipeline.main import detect_ms1_features

detect_ms1_features(
    path = "C:/project_path_namet/"
)
```

### 5b. For TMT: Reporter Ion Quantification
```python
from proteomics_pipeline.main import tmt_quant_function

tmt_quant_function(
    path = "C:/project_path_name/",
    da_tol = 0.003
)
```

### 6. FDR Control and Result Processing
```python
from proteomics_pipeline.main import train_fdr_model

results = train_fdr_model(
    path = "C:/project_path_name/",
    fasta = None,  # Will use fasta file in path
    lfq_tmt = 'tmt',
    psm_fdr = 0.01,
    peptide_fdr = 0.01,
    protein_fdr = 0.01
)
```

## Command Line Interface
The pipeline provides several CLI commands:
```
pride_repo_file_display    # Output list files from PRIDE repository
internal_repo_file_display # Output list files from internal repository
raw_file_extract          # Download/copy instument raw files
raw_to_mzml               # Convert instrument raw to mzML format
db_search                 # Run database search
detect_ms1_features       # Run MS1 feature detection
tmt_quant_function        # Quantify TMT reporter ions
train_fdr_model          # Process results with FDR control
```

## Input Requirements
<ul>
    <li>Raw mass spectrometry files (.raw)</li>
    <li>FASTA database file for searches</li>
    <li>Comet parameter file (or use built-in defaults)</li>
    <li>Project directory structure</li>
</ul>

## Output Files
The pipeline generates:
<ul>
    <li>Processed mzML files in path/raw/{eg .raw files}</li>
    <li>Processed mzML files in path/mzml/{.mzml files}</li>
    <li>Database search results in path/mzml/{.pin files}</li>
    <li>Feature quantification tables:
        <ul>
            <li>LFQ: path/ms1_features/{.features.tsv files}</li>
            <li>TMT: path/ms[2/3]_features/{.features.tsv files}</li>
        </ul>
    </li>
    <li>FDR-controlled results:
        <ul>
            <li>Combined results file with specified FDR thresholds in path/mokapot_results/{.csv files}</li>
            <li>Mokapot model file in path/mokapot_results/{svc.model}</li>
        </ul>
    </li>
    <li>Log files for each processing step in path/logs/{.log files}</li>
</ul>

## Function Documentation
Each function's documentation can be accessed using Python's help() function:

```python
help(pride_repo_file_display)
help(internal_repo_file_display)
help(raw_file_extract)
help(raw_to_mzml)
help(db_search)
help(detect_ms1_features)
help(tmt_quant_function)
help(train_fdr_model)
```

