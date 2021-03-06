# Test run with the ECC Sculptures dataset

Canonical URL of the dataset: <http://semantics.gr/authorities/vocabularies/ecc-sculptures-dataset>

## Downloading the dataset description using the crawler tool

```bash
docker-compose run --rm --user 1000:1000 crawl starter.sh  \
   --dataset-uri http://semantics.gr/authorities/vocabularies/ecc-sculptures-dataset \
   --output ecc-sculptures.nt
```

## Validating the dataset description

Before processing the complete data we validated the dataset description. We used a SHACL shape file defined for datasets with URI lists using the VOID ontology:  

```bash
docker-compose run --rm --user 1000:1000 validate starter.sh \
  --data ecc-sculptures.nt \
  --shape shacl_dataset_list_void.ttl \
  --output ecc-sculptures-val-ds.ttl
```

The validation was succesful, see the ecc-sculptures-val-ds.ttl report.

## Mapping the schema.org data to EDM data

Next step was doing the actual conversion of the resources.

```bash
docker-compose run --rm --user 1000:1000 map starter.sh \
  --data ecc-sculptures.nt \
  --query schema2edm.rq \
  --format RDF/XML \
  --output ecc-sculptures-edm.rdf
```

Note: in `.env` VAR_PROVIDER was set to 'ECC'.

## Validate the total the EDM data

Next the generated EDM data was validated.

```bash
docker-compose run --rm --user 1000:1000 validate starter.sh \
  --data ecc-sculptures-edm.rdf \
  --shape shacl_edm.ttl \
  --output ecc-sculptures-edm-val.ttl
```

The validation resulted in no errors for the sculptures dataset.

## Zip the result for transport to Europeana

```bash
# note the ./data in this command!
gzip ./data/ecc-sculptures-edm.rdf
```

The result file will be created in the `data` dir and is called `ecc-sculptures-edm.rdf.gz`

All the files generated by this test run are stored in the `test` dir under `ecc-sculptures` for documentation purposes.
