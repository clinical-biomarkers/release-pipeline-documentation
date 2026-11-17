# release-pipeline-documentation
A-to-Z documentation for data releases

## Pipeline Steps

- **Backend API Release Pipeline:** [link](https://github.com/clinical-biomarkers/backend-pipeline/blob/main/README.md)
  - *Note:* documentation for Step-1 - downloading raw input files and transforming them into `compiled` datasets (only the most recent ones have been documented) [here](https://github.com/clinical-biomarkers/biomarker-extraction)
- **Data Release Pipeline (Data Portal):** [link](https://github.com/clinical-biomarkers/biomarker-data-release-pipeline/blob/master/README.md)

## Maintenance Checklist

### When new data is downloaded
1. Properly formatted TSV files must be placed into the Downloads directory (`/data/shared/biomarkerdb/downloads/<data_source>/current/<data_source>.tsv`). Currently they are either already present there, or being output from the `biomarker-extraction` pipeline.
   ```
   # Replace <data_source> with the name of the source where the new data comes from (use snake_case)
   mkdir -p /data/shared/biomarkerdb/downloads/<data_source>/<yyyy_mm_dd> # yyyy_mm_dd is today's date
   # Create the "current" symlink to today's directory
   ln -s $(date +%Y_%m_%d) /data/shared/biomarkerdb/downloads/<data_source>/current
   ```
2. Add the file name to `/data/shared/biomarkerdb/releases/data/current/misc/datasets.json` under `"biomarkers"`.
   ```
   ,"biomarkers_<data_source>":["<data_source>.tsv"]
   ```

### When a new ID space with a URL is introduced
Add a row to the xrefs file: `/data/shared/biomarkerdb/releases/data/current/misc/xref_info.csv`
- Columns: *`xref_key | xref_lbl | xref_url | xref_id_examples`*
  - `xref_key`: use snake_case
  - `xref_lbl`: use proper casing that the data source uses, e.g. CIViC, UniProtKB
  - `xref_url`: URL template used for linking to the entity page on the data source website. Use `%s` as ID placeholder
  - `xref_id_examples`: example of an ID (that replaces `%s` in the actual URL)
- Example row for `assessed_biomarker_entity_id` ID space
  ```
  entity_xref_civic,CIViC,https://civicdb.org/molecular-profiles/%s,447
  ```
- Example row for a non-entity ID space, e.g. `condition_id`
  ```
  doid,Disease Ontology,http://purl.obolibrary.org/obo/DOID_%s,123
  ```
## KG Release Pipeline
1. Convert `biomarkerdb` JSON objects to the NT format using [format-converter](https://github.com/clinical-biomarkers/format-converter)
   ```
   cd /data/shared/repos/format-converter
   source env/bin/activate
   python main.py /data/shared/biomarkerdb/releases/data/current/jsondb/biomarkerdb /data/shared/biomarkerdb/downloads/nt/current/all-biomarkers.nt
   ```
2. Convert NT files to the OWL format using [nt-owlnets-kg-converter](https://github.com/clinical-biomarkers/nt-owlnets-kg-converter)
   ```
   cd /data/shared/repos/nt-owlnets-kg-converter
   source env/bin/activate
   python main.py /data/shared/biomarkerdb/downloads/nt/current/all-biomarkers.nt
   ```
