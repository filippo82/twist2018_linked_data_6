List of SPARQL queries
======================

Endpoint
--------
[https://ld.stadt-zuerich.ch/query](https://ld.stadt-zuerich.ch/query)

### Area per quarter (area: only land without forest, in km2)
```PREFIX qb: <http://purl.org/linked-data/cube#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dataset: <https://ld.stadt-zuerich.ch/statistics/dataset/>
PREFIX measure: <https://ld.stadt-zuerich.ch/statistics/measure/>
PREFIX dimension: <https://ld.stadt-zuerich.ch/statistics/property/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX code: <https://ld.stadt-zuerich.ch/statistics/code/>

SELECT ?spaceLabel ?year ?areaKm2
WHERE{
  GRAPH <https://linked.opendata.swiss/graph/zh/statistics>{

    ?obsarea a qb:Observation ;
      qb:dataSet dataset:STF-RAUM-ZEIT-BBA ;
      measure:STF ?area ;
      dimension:BBA code:BBA1000;
      dimension:RAUM ?space ;
      dimension:ZEIT ?time .

   ?space rdfs:label ?spaceLabel.     
   BIND(SUBSTR(STR(?space),45,4) AS ?spacegroup)
   BIND((?area / 100) AS ?areaKm2)
   BIND(STR(?time) AS ?year)    
   FILTER (?spacegroup  IN ('R000', 'R001')) #quarters only
   FILTER(?time = "2017-12-31"^^xsd:date)     

  }}

ORDER BY ?spaceLabel ?year```
