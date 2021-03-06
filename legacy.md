# Legacy Documentation

The following material may be useful to understand the current build procedure and contains 
legacy documentation on several procedures and Dockstore's architecture. 

## For Dockstore Developers

### Swagger Java Client for Dockstore

This will no longer be necessary to do manually and is now done as part of the Maven build process.
Just remember to commit a new `dockstore-webservice/src/main/resources/swagger.yaml` when the dockstore API changes. 
Content is left here for reference purposes. 

Background:

 * Client library generated by the [swagger code generator](https://github.com/swagger-api/swagger-codegen)
 * Is generated based on the webservice's swagger UI spec
 * Used by the Dockstore CLI to make http requests to the dockstore
 * If you changed/added some endpoints that the CLI uses, you will need to regenerate the swagger client.
 
To regenerate the swagger client:

1. Have the dockstore webservice running
2. Pull the code from their repo and cd to the directory. We are using v2.1.4. Build using `mvn clean install`
3. Run `java -jar modules/swagger-codegen-cli/target/swagger-codegen-cli.jar generate -i http://localhost:8080/swagger.json -l java -o <output directory> --library jersey2`. The output directory is where you have dockstore/swagger-java-client/.
4. NOTE: Re-generating the swagger client will probably generate an incorrect pom file. Use git checkout on the pom file to undo the changes to it.

### Swagger Java Client for quay.io

This will no longer be necessary to do manually and is now done as part of the Maven build process.
Content is left here for reference purposes. 

Background:

 * Client library generated by the [swagger code generator](https://github.com/swagger-api/swagger-codegen)
 * Is generated based on the quay.io's swagger UI spec
 * Used by the Dockstore CLI to make http requests to quay.io
 * If CoreOS changes their API, you will need to regenerate the swagger client.
 
 To regenerate the swagger client:
 
1. Run `echo "{\"library\":\"jersey2\",\"invokerPackage\":\"io.swagger.quay.client\",\"modelPackage\":\"io.swagger.quay.client.model\",\"apiPackage\":\"io.swagger.quay.client.api\"}" > config.json`
2. Run `java -jar modules/swagger-codegen-cli/target/swagger-codegen-cli.jar generate -i https://quay.io/api/v1/discovery -l java -o <output directory> --library jersey2 --config config.json`. The output directory is where you have dockstore/swagger-java-client/.
3. NOTE: Rengenerating the swagger client will probably generate an incorrect pom file. Use git checkout on the pom file to undo the changes to it.

### CWL Avro documents

Background:
* The CWL specification is defined in something similar to but not entirely like Avro
* Use the schema salad project to convert to an avro-ish schema document
* Generate the Java classes for the schema
* We cannot use these classes directly since CWL documents are not json or avro binaries, use cwl-tool to convert to json and 
then gson to convert from json due to some incompatibilities between CWL avro and normal avro.  

To regenerate:

1. Get schema salad from the common-workflow-language organization and run `python -mschema_salad --print-avro ~/common-workflow-language/draft-3/cwl-avro.yml`
2. Get the avro tools jar and CWL avsc and call `java -jar avro-tools-1.7.7.jar compile schema cwl.avsc cwl`
3. Copy them to the appropriate directory in dockstore-client (you will need to refactor and insert package names)

Eventually, this will be moved out as a proper Maven dependency on https://github.com/common-workflow-language/cwlavro
