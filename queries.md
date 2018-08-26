List of SPARQL queries
======================

Endpoint
--------
[https://ld.stadt-zuerich.ch/query](https://ld.stadt-zuerich.ch/query)

### Area per quarter (area: only land without forest, in km2)
```
PREFIX qb: <http://purl.org/linked-data/cube#>
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

ORDER BY ?spaceLabel ?year
```

### Proportion of green area (parks, forest, soccer fields, graveyards) per quarter
```
PREFIX qb: <http://purl.org/linked-data/cube#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dataset: <https://ld.stadt-zuerich.ch/statistics/dataset/>
PREFIX measure: <https://ld.stadt-zuerich.ch/statistics/measure/>
PREFIX dimension: <https://ld.stadt-zuerich.ch/statistics/property/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX code: <https://ld.stadt-zuerich.ch/statistics/code/>

SELECT ?spaceLabel ?year (SUM(?greenarea/?totalarea * 100) AS ?greenAreaRatio)
WHERE{
  GRAPH <https://linked.opendata.swiss/graph/zh/statistics>{
    {
    SELECT ?spaceLabel ?year (SUM(?totalarea) AS ?totalarea)
    WHERE{
    ?obsarea a qb:Observation ;
      qb:dataSet dataset:STF-RAUM-ZEIT-BBA ;
      measure:STF ?totalarea ;         
      dimension:RAUM ?space ;
      dimension:ZEIT ?time .    
      {?obsarea a qb:Observation ;
        qb:dataSet dataset:STF-RAUM-ZEIT-BBA ;
        dimension:BBA code:BBA1000.} #land without forest
        UNION
      {?obsarea a qb:Observation ;
        qb:dataSet dataset:STF-RAUM-ZEIT-BBA ;
        dimension:BBA code:BBA2000.} #forest
        UNION
      {?obsarea a qb:Observation ;
        qb:dataSet dataset:STF-RAUM-ZEIT-BBA ;
        dimension:BBA code:BBA3000.}  #waterbodies

     ?space rdfs:label ?spaceLabel.            
     BIND(SUBSTR(STR(?space),45,4) AS ?spacegroup)
     FILTER (?spacegroup  IN ('R000', 'R001')) #quarters only
     FILTER(?time = "2017-12-31"^^xsd:date)          
     BIND(STR(?time) AS ?year)            
     }    
     GROUP BY ?spaceLabel ?year
     ORDER BY ?spaceLabel ?year     
    }


    {
    SELECT ?spaceLabel ?year (SUM(?greenarea) AS ?greenarea)
    WHERE{
    ?obsarea a qb:Observation ;
      qb:dataSet dataset:STF-RAUM-ZEIT-BBA ;
      measure:STF ?greenarea ;         
      dimension:RAUM ?space ;
      dimension:ZEIT ?time .    
      {?obsarea a qb:Observation ;
        qb:dataSet dataset:STF-RAUM-ZEIT-BBA ;
        dimension:BBA code:BBA1220.} #parks, sport areas, graveyard
        UNION
      {?obsarea a qb:Observation ;
        qb:dataSet dataset:STF-RAUM-ZEIT-BBA ;
        dimension:BBA code:BBA1420.} #meadows, fields
        UNION
      {?obsarea a qb:Observation ;
        qb:dataSet dataset:STF-RAUM-ZEIT-BBA ;
        dimension:BBA code:BBA2000.}  #forest

     ?space rdfs:label ?spaceLabel.            
     BIND(SUBSTR(STR(?space),45,4) AS ?spacegroup)
     FILTER (?spacegroup  IN ('R000', 'R001')) #quarters only
     FILTER(?time = "2017-12-31"^^xsd:date)          
     BIND(STR(?time) AS ?year)            
     }    
     GROUP BY ?spaceLabel ?year
     ORDER BY ?spaceLabel ?year     
    }

  }}

  GROUP BY ?spaceLabel ?year
  ORDER BY ?spaceLabel ?year
```
