# Test run with the ECC photographs dataset

Canonical URL of the dataset: <http://semantics.gr/authorities/vocabularies/ecc-photographs-dataset>

## Downloading the dataset description using the crawler tool

Currently the crawler doesn't support a straight download of the dataset yet. Instead we used the following work around to download the complete dataset (including the dataset description). The following 'curl' command was run in the home dir of the lod-aggregator:

```bash
# note the ./data/ in this command!
curl -H "Accept: application/rdf+xml" \
     https://www.semantics.gr/authorities/vocabularies/ecc-photographs-dataset \
     > ./data/ecc-photographs.rdf
```

## Validating the dataset description

Before processing the complete data we validated the dataset description. We used a SHACL shape file defined for datasets with URI lists using the VOID ontology:  

```bash
docker-compose run --rm --user 1000:1000 validate starter.sh \
  --data ecc-photographs.rdf \
  --shape shacl_dataset_list_void.ttl \
  --output ecc-photographs-val-ds.ttl
```

The validation was succesful, see the gr_validate_dataset.ttl report.

## Mapping the schema.org data to EDM data

Next step was doing the actual conversion of the resources.

```bash
docker-compose run --rm --user 1000:1000 map starter.sh \
  --data ecc-photographs.rdf \
  --query schema2edm.rq \
  --format RDF/XML \
  --output ecc-photographs-edm.rdf
```

Note: in `.env` VAR_PROVIDER was set to 'ECC'.

## Validate the total the EDM data

Next the generated EDM data was validated.

```bash
docker-compose run --rm --user 1000:1000 validate starter.sh \
  --data ecc-photographs-edm.rdf \
  --shape shacl_edm.ttl \
  --output ecc-photographs-edm-val.ttl
```

The validation resulted in no errors for the photographs dataset.

## Zip the result for transport to Europena

```bash
# note the ./data in this command!
gzip ./data/ecc-photographs-edm.rdf
```

The result file will be created in the `data` dir and is called `ecc-photographs-edm.rdf.gz`

All the files generated by this test run are stored in the `test` dir under `ecc-photographs` for documentation purposes.