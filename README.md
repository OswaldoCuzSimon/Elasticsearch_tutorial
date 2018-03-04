# ElasticSearch Guide
### Installing Elasticsearch
#### Install Elasticsearch with Docker
~~~bash
docker run -d -p 9200:9200 \
-e "http.host=0.0.0.0" \
-e "transport.host=127.0.0.1" \
-e "xpack.security.enabled=false" \
-e "network.bind_host=0.0.0.0" \
docker.elastic.co/elasticsearch/elasticsearch:6.2.2
~~~

### Index operations

Para crear un indice
`curl -XPUT 'localhost:9200/platzi?pretty'`
Para obtener informacion de un indice
`curl -XGET 'localhost:9200/platzi?pretty'`
Para borrrar un indice
`curl -XDELETE 'localhost:9200/platzi?pretty'`
*Nota: el parametro pretty es para que la respuesta este identada*
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
Asi que si necesitas todos debes indicarle cuantos son una forma es preguntar primero
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

### Delete documents
Borrar un solo documento:
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
~~~~


