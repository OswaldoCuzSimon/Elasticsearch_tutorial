# ElasticSearch Guide

### Indexing
Para crear un documento se hace con PUT a la url de Elasticsearch + index + type + id, pero el id no es necesario si no se informa sera generado por Elasticsearch.
*Nota: el parametro pretty es para que la respuesta este identada*
~~~
curl -XPUT 'localhost:9200/test/dummy/1?pretty' -H 'Content-Type: application/json' -d'
{
    "user" : "Oswaldo",
    "post_date" : "2018-11-15T14:12:12",
    "message" : "Hello platzi"
}
'
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


