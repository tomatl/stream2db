# Stream2db

Stream json documents or a csv file to a backend.
Currently we support `mongodb` and `elasticsearch`. More
backends could be added easily using the [node etl driver](https://github.com/ZJONSSON/node-etl).

## Examples

### Import compranet from streaming source to elasticsearch.

Since [ellison](https://github.com/kyv/ellison) hashes documents before sending them over the wire, those streams will get checked for data corruption.

    node app.js https://excel2json.herokuapp.com/https://compranetinfo.funcionpublica.gob.mx/descargas/cnet/Contratos2013.zip

### Use *CODIGO_CONTRATO* as _id

If you do not provide an *ID* field (`--id`) a random *ID* will be generated. If you do set the *ID* new documents with the same *ID* will replace their predecessors.

    node app.js -i CODIGO_CONTRATO https://excel2json.herokuapp.com/https://compranetinfo.funcionpublica.gob.mx/descargas/cnet/Contratos2013.zip

### Import CSV into *cargografias* index on elasticsearch

You can use a csv file as your data source.

    node app.js -d cargografias ~/Downloads/Cargografias\ v5\ -\ Nuevos_Datos_CHEQUEATON.csv

## Options

You can set some options on the commandline.

    node app.js -h|--help
    --backend DATA BACKEND   Backend to save data to. [mongo|elastic]
    --db INDEX|DB            Name of the index (elastic) or database (mongo) where data is written
    --type TYPE|COLLECTION   Mapping type (elastic) or collection (mongo).
    --id ID                  Specify a field to be used as _id. If hash is specified the object hash will be used
    --uris URIS              Space seperated list of urls to stream
    --host HOST              Host to stream to. Default is localhost
    --port PORT              Port to stream to. Defaults to 9500 (elastic) or 27017 (mongo)
    --help                   Print this usage guide.

## Debugging

The `--verbose` flag triggers debugging mode of the DB driver. En elasticsearch this is set to `log: trace`. The mongo driver allows for configuration by way of [variables in the enviornment](https://automattic.github.io/monk/docs/Debugging.html).

## Notes

As we are targeting local data managment, we have not yet added DB authorization. This will get added to the parameters.

## Cleanup

strings are [normalized](https://www.npmjs.com/package/normalize-space) and [trimmed](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/String/trim).

### Type coercion

We do very simple type coercion. Numbers should work.

elasticsearch indexes (processes) data as it is inserted. So data types must be set in the mapping before data is inserted. It has dynamic types which will try to detect types (numbers, dates, etc) but the format of the data must conform to what is expected.

In mongodb we can create the index after the data is inserted, so we leave that up to you.

### Hashes and Duplicates

We detect and dismiss duplicates using [object-hash](https://github.com/puleos/object-hash). We also add the field `hash` to the indexed document.