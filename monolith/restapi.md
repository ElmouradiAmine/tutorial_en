# Tutorial on microservices with JHipster :: Utilisation de l'API REST avec cURL

Run the following commands

```bash

# Installation
sudo apt-get install curl jq
sudo npm install -g jwt-cli

# Content-Type
ACCEPT_JSON="Accept: application/json"
ACCEPT_CSV="Accept: text/csv"
CONTENT_JSON="Content-Type: application/json"
CONTENT_CSV="Content-Type: text/csv"

# LOCAL
PORT=8080
URL=http://localhost:$PORT

# PROD
#PORT=443
#URL=https://storeNOMPRENOM.VOTRE_NOM_PRENOM.com:$PORT
#URL=https://storeNOMPRENOM.herukoapp.com:$PORT

# Doc
URL_APIDOC=${URL}/v2/api-docs

# Operations
# CURL="curl --verbose"
CURL="curl -k --verbose"
GET="${CURL} -X GET --header \""$ACCEPT_JSON"\""
POST="${CURL} -X POST --header \""$ACCEPT_JSON"\""
PUT="${CURL} -X PUT --header \""$ACCEPT_JSON"\""
DELETE="${CURL} -X DELETE --header \""$ACCEPT_JSON"\""
OPTIONS="${CURL} -X OPTIONS --header \""$ACCEPT_JSON"\""
HEAD="${CURL} -X HEAD --header \""$ACCEPT_JSON"\""

# ===================================
# Get OpenAPI2.0 specification of the API
# -----------------------------------
${GET} ${URL_APIDOC} > swagger.json

# ===================================
# Authentication operations
# -----------------------------------
USERNAME=user
PASSWORD=user

AUTH_JSON="{ \"username\": \"${USERNAME}\", \"password\": \"${PASSWORD}\" }"

# Get the Bearer token for the user
rm $USERNAME.token.json
${POST}  --header "$CONTENT_JSON" -d "$AUTH_JSON" ${URL}/api/authenticate > $USERNAME.token.json
TOKEN=$(jq -r '.id_token' $USERNAME.token.json)
AUTH="Authorization: Bearer $TOKEN"

# Decode JWT Token for fun !
echo "Decode JWT Token $TOKEN"
jwt $TOKEN

# Get account info
${GET} --header "$AUTH" ${URL}/api/account

# ===================================
# Operations on Product resource
# -----------------------------------

# Get all products
${GET} ${URL}/api/products
# --> 401 Unauthorized
${GET} --header "$AUTH" ${URL}/api/products
# --> 200
${GET} --header "$AUTH" ${URL}/api/products/1

# Add one new product
PRODUCT='{
  "name": "TEST",
  "description": "This is a test",
  "price": 5,
  "size": "XXL",
  "productCategory": {
    "id": 1,
    "name": "TEST",
    "description": "TESTTEST"
  }
}'
${POST} --header "$AUTH" --header "$CONTENT_JSON" ${URL}/api/products -d "$PRODUCT"

# Get the id of the created product
# TODO with jq ".id"
PRODUCT_ID=3

# Get all products
${GET} ${URL}/api/products

# Update one existing product
CHANGE='{
  "price": 1500
}'
${PUT} --header "$AUTH" --header "$CONTENT_JSON" ${URL}/api/products/${PRODUCT_ID} -d "$CHANGE"
# --> 400

# Update one existing product
CHANGE='{
  "name": "NEWTEST"
  "price": 1500,
  "size": "S",
}'
${PUT} --header "$AUTH" --header "$CONTENT_JSON" ${URL}/api/products/${PRODUCT_ID} -d "$CHANGE"
# --> according the PUT meaning, other properties should not be updated by the operation

# Get the updated product
${GET} --header "$AUTH" --header "$CONTENT_JSON" ${URL}/api/products/${PRODUCT_ID}

# Get all products
${GET} --header "$AUTH" --header "$CONTENT_JSON" ${URL}/api/products

# Remove the updated product
${DELETE} --header "$AUTH" --header "$CONTENT_JSON" ${URL}/api/products/${PRODUCT_ID}

```

## Generation of the REST API
The service descriptor is available here http://localhost:8080/v2/api-docs .

It is generated from the annotations on the classes `Resource` of the package `com.mycompany.store.web.rest`and the Entity or DTO classes (if the [`dto` directive](https://www.jhipster.tech/jdl/) is selected).

You can add [Swagger annotations](https://github.com/swagger-api/swagger-core/wiki/Annotations-1.5.X) in order to improve the API.

Install `swagger-codegen` ([More information](https://swagger.io/docs/open-source-tools/swagger-codegen/)).
```bash
mkdir -p ~/github/mastering-microservices/swagger-codegen
cd ~/github/mastering-microservices/swagger-codegen
wget https://oss.sonatype.org/content/repositories/releases/io/swagger/swagger-codegen-cli/2.2.1/swagger-codegen-cli-2.2.1.jar
mv swagger-codegen-cli-2.2.1.jar swagger-codegen.jar
java -jar swagger-codegen.jar help
```
> Available languages: [android, aspnet5, async-scala, cwiki, csharp, cpprest, dart, flash, python-flask, go, groovy, java, jaxrs, jaxrs-cxf, jaxrs-resteasy, jaxrs-spec, inflector, javascript, javascript-closure-angular, jmeter, nancyfx, nodejs-server, objc, perl, php, python, qt5cpp, ruby, scala, scalatra, silex-PHP, sinatra, rails5, slim, spring, dynamic-html, html, html2, swagger, swagger-yaml, swift, tizen, typescript-angular2, typescript-angular, typescript-node, typescript-fetch, akka-scala, CsharpDotNet2, clojure, haskell, lumen, go-server]

```bash
codegen() {
  mkdir -p $1
  (cd $1;  java -jar ../swagger-codegen.jar generate -i ../swagger.json -l $1)  
}
```

Get the Swagger definition. You can install the tool `jq` (https://stedolan.github.io/jq/download/) for formatting the document `swagger.json`.
```bash
cd ~/github/mastering-microservices/swagger-codegen
wget http://localhost:8080/v2/api-docs -O swagger.json
jq '.' swagger.json
```

> Remark: A plugin [Swagger Codegen](https://github.com/swagger-api/swagger-codegen/tree/master/modules/swagger-codegen-maven-plugin) for Maven is available.

### Generate the HTML documentation

```bash
codegen html2
(cd html2; open index.html)
```

### Generate the clients
```bash
# Client bash basé sur cURL
# codegen bash; (cd bash; tree . ; cloc .)

codegen typescript-angular; (cd typescript-angular; tree . ; cloc .)

codegen python; (cd python; tree . ; cloc .)

codegen cpprest; (cd cpprest; tree . ; cloc .)

codegen php; (cd php; tree . ; cloc .)

```

#### Generate the server skeletons
```bash
codegen python-flask; (cd python-flask; tree . ; cloc .)

codegen nodejs-server; (cd nodejs-server; tree . ; cloc .)

codegen spring; (cd spring; tree . ; cloc .)

codegen go-server; (cd go-server; tree . ; cloc .)
```

> Remarque:  la génération du squelette du serveur Pistache C++ n'est disponible que via

### Generate the injection plan for [Apache JMeter](https://jmeter.apache.org/)
```bash
codegen jmeter; (cd jmeter; ls -al)
```
> Rien pour l'injecteur de charge [Gatling](https://gatling.io/) !
