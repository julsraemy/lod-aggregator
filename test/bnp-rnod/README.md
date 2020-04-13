# Test run with the Registo Nacional de Obras Digitais (RNOD) - National Library of Portugal

Canonical URL of the dataset: <http://oai.bn.pt/conjunto_de_dados/rnod-europeana.rdf>

## Downloading the dataset description using the crawler tool

```bash
docker-compose run --rm --user 1000:1000 crawl starter.sh \
  --dataset-uri http://oai.bn.pt/conjunto_de_dados/rnod-europeana.rdf \
  --description-only \
  --output rnod-description.nt
```

The dataset description was downloaded without errors.

## Validating the dataset description

Before processing the complete data we validated the dataset description. We used a SHACL shape file defined for datasets with a dump as distribution method described in the Schema.org ontology:  

```bash
docker-compose run --rm --user 1000:1000 validate starter.sh \
  --data rnod-description.nt \
  --shape shacl_dataset_dump_schema.ttl \
  --output rnod-description-val-ds.ttl
```

The validation was succesful, see the rnod-description-val-ds.ttl report.

## Downloading the complete dataset using the crawler tool

```bash
docker-compose run --rm --user 1000:1000 crawl starter.sh \
  --dataset-uri http://oai.bn.pt/conjunto_de_dados/rnod-europeana.rdf \
  --output rnod.nt
```

The download completed succesfully but the crawler found 721 errors: _"Bad character in IRI (space)"_.
See `crawler.log` for more details.

## Fix the downloaded data

Because the downloaded RDF is already in EDM format no mapping is neccessary. So we can run a check on the downloaded data. But the validator crashes when running with on the `rnod.nt` datafile, probably caused by te URI problems we saw when processing the downloading the file.  

A workaround was to convert the downloaded N-triples file to a RDF/XML serialization using the following command:

```bash
docker-compose run --rm --user 1000:1000 serialize starter.sh \
  --data rnod.nt \
  --format RDF/XML \
  --output rnod.ttl
```

Altough this generates warnings and an error it does produce a valid RDF/XML file which we can process.

## Validate the EDM data

```bash
docker-compose run --rm --user 1000:1000 validate starter.sh \
  --data rnod.rdf \
  --shape shacl_edm.ttl \
  --output rnod-edm-val.ttl
```

The validation resulted in number of errors, see `rnod-edm-val.ttl` for a complete report. In order to get a clearer overview of the errors we can list the errors to a CSV file with the following command:

```bash
docker-compose run --rm --user 1000:1000 map starter.sh \
  --data rnod-edm-val.ttl \
  --query list-errors.rq \
  --format CSV \
  --output errors.csv
```

The main problems found are resources without an `edm:rights` property and (some SHACL debugging was neccessary here) resources without a `edm:shownBy` or `edm:shownAt` property.

## Zip the result for transport to Europena

```bash
# note the ./data in this command!
gzip ./data/rnod.rdf
```

The result file will be created in the `data` dir and is called `rnod.rdf.gz`

All the files generated by this test run are stored in the `test` dir under `bnp-rnod` for documentation purposes. The orginal dump was deleleted because of the large size.