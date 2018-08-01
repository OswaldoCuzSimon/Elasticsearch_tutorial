# ElasticSearch Guide
### Installing Elasticsearch
#### Install Elasticsearch with Docker
~~~
docker run -d -p 9200:9200 \
-e "http.host=0.0.0.0" \
-e "transport.host=127.0.0.1" \
-e "xpack.security.enabled=false" \
-e "network.bind_host=0.0.0.0" \
docker.elastic.co/elasticsearch/elasticsearch:6.2.2
~~~

### Index operations

Para crear un indice
~~~
curl -XPUT 'localhost:9200/platzi?pretty'
~~~
Para obtener informacion de un indice
~~~
curl -XGET 'localhost:9200/platzi?pretty'
~~~
Para borrrar un indice
~~~
curl -XDELETE 'localhost:9200/platzi?pretty'
~~~
*Nota: el parametro pretty es para que la respuesta este identada, prueba sin `?pretty`*

Listar todos los indices
~~~
curl -X GET "localhost:9200/_cat/indices?v"
~~~

### Indexing documents
Para crear un documento se hace con PUT a la url de Elasticsearch + index + type + id, pero el id no es necesario si no se informa sera generado por Elasticsearch.

~~~
curl -XPUT 'localhost:9200/test/dummy/1?pretty' -H 'Content-Type: application/json' -d'
{
    "user" : "Oswaldo",
    "post_date" : "2018-11-15T14:12:12",
    "message" : "Hello platzi"
}
'
~~~
### Get documents
para obtener un solo documento

~~~
curl -XGET 'localhost:9200/test/dummy/1?pretty' -H 'Content-Type: application/json'
~~~
Para saber cuantos documentos hay en un type
~~~
curl -XGET 'localhost:9200/test/dummy/_count?pretty'
~~~

Elasticsearch no puede traer todos los documentos en un type por default solo trae 10
Asi que si necesitas todos debes indicarle cuantos son, una forma de traer todos es preguntar primero
cuantos hay con la query de arriba o indicando un size que de antemano se sabe que
sobre pasa el numero de documentos en el type
Para obtener *n* documentos
~~~
curl -XGET 'localhost:9200/test/dummy/_search?size=100&pretty' -H 'Content-Type: application/json' -d'
{
	"query":{
	"match_all":{} }
}'
~~~
*Nota: si se elimina `size=100&` solo traera 10*
Lo mismo pero mas corto
`curl -XGET 'localhost:9200/test/dummy/_search?size=100&pretty'`
### Update documents
Actualizar solo una parte del documento
~~~
curl -XPOST 'localhost:9200/test/dummy/1/_update?pretty' -H 'Content-Type: application/json' -d'
{
    "doc" : {
        "last_name" : "Cruz"
    }
}
'
~~~
Para actualizar todo el documento basta con repetir [query](#indexing-documents) pero con datos diferentes
### Delete documents
Borrar un solo documento
~~~
curl -XDELETE 'localhost:9200/test/dummy/1?pretty'
~~~
Borrar todos los documentos en un type
~~~
curl -XPOST 'localhost:9200/test/dummy/_delete_by_query?conflicts=proceed&pretty' -d'
{
   "query": {
       "match_all": {}
   }
}'
~~~
### Bulk
Bulk permite hacer multiples acciones (operaciones) de creacion, actualizacion o borrado en una sola llamada a la API, esto mejora la velocidad que si se hacen varias llamadas.

Ejemplo de inserción, borrado y actualización:
~~~
curl -XPOST 'localhost:9200/_bulk?pretty' -H 'Content-Type: application/json' -d'
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "_doc", "_id" : "2" } }
{ "update" : {"_id" : "1", "_type" : "_doc", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
'
~~~
Para creacion y actualizacion se necesita dos renglones uno con la acccion y otro con los datos, para borrar solo se necesita la acción, es importante que cada accion y documento esten en una solo linea y no tengan coma al final tambien es importante el un salto de linea despues de la ultima accion

Cargar datos de ejemplo a Elasticsearch:
~~~~
curl -XPOST 'localhost:9200/_bulk?pretty' -H 'Content-Type: application/json' --data-binary @scikit_aportes.json
curl -XPOST 'localhost:9200/_bulk?pretty' -H 'Content-Type: application/json' --data-binary @PostgreSQL_aportes.json
curl -XPOST 'localhost:9200/_bulk?pretty' -H 'Content-Type: application/json' --data-binary @keyword_data.json
~~~~
### Update by query
actualizara los documentos que tengan en el campo user el valor GOLLUM23, no actualizara si encuentra GOLLUM23 en el campo user dentro del arreglo de replies, por lo que solo encontrara una coincidencia
~~~
curl -XPOST 'localhost:9200/platzi/_update_by_query?pretty' -H 'Content-Type: application/json' -d'
{
    "script": {
        "inline": "ctx._source.user='\''GOLLUM23 2.0'\''"
    },
    "query": {
        "match": {
            "user": "GOLLUM23"
        }
    }
}'
~~~
*Nota: `'\''` Es para escapar la comilla simple `'`*

Para evitar escapar las comillas se puede mandar el parametro data mediante archivo.
~~~
curl -XPOST 'localhost:9200/platzi/_update_by_query?pretty' -H 'Content-Type: application/json' \
--data-binary @update_by_query_example.json
~~~
