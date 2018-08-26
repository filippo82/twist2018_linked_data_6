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

### Population density per quarter (people per km2 (land without forest)
```
PREFIX qb: <http://purl.org/linked-data/cube#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dataset: <https://ld.stadt-zuerich.ch/statistics/dataset/>
PREFIX measure: <https://ld.stadt-zuerich.ch/statistics/measure/>
PREFIX dimension: <https://ld.stadt-zuerich.ch/statistics/property/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX code: <https://ld.stadt-zuerich.ch/statistics/code/>

SELECT ?spaceLabel ?year ?peoplePerKm2
WHERE{
  GRAPH <https://linked.opendata.swiss/graph/zh/statistics>{
    ?obspop a qb:Observation ;
      qb:dataSet dataset:BEW-RAUM-ZEIT ;     
      measure:BEW ?population ;
      dimension:RAUM ?space ;
      dimension:ZEIT ?time .

    ?obsarea a qb:Observation ;
      qb:dataSet dataset:STF-RAUM-ZEIT-BBA ;
      measure:STF ?area ;
      dimension:BBA code:BBA1000;
      dimension:RAUM ?space ;
      dimension:ZEIT ?time .

   ?space rdfs:label ?spaceLabel.     
   BIND(SUBSTR(STR(?space),45,4) AS ?spacegroup)
   BIND((?population/?area * 100) AS ?peoplePerKm2)
   BIND(STR(?time) AS ?year)    
   FILTER (?spacegroup  IN ('R000', 'R001')) #quarters only
   FILTER(?time = "2017-12-31"^^xsd:date)     

  }}
ORDER BY ?year
```

### 80 years and older, 15 years or younger
```
PREFIX qb: <http://purl.org/linked-data/cube#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dataset: <https://ld.stadt-zuerich.ch/statistics/dataset/>
PREFIX measure: <https://ld.stadt-zuerich.ch/statistics/measure/>
PREFIX dimension: <https://ld.stadt-zuerich.ch/statistics/property/>
PREFIX code: <https://ld.stadt-zuerich.ch/statistics/code/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT ?spaceLabel ?year  
	(SUM(?age80Plus/?population *100) AS ?age80PlusRatio)
	(SUM(?age15Minus/?population *100) AS ?age15MinusRatio)

WHERE{
  GRAPH <https://linked.opendata.swiss/graph/zh/statistics>{   

   ?obpop a qb:Observation ;
      qb:dataSet dataset:BEW-RAUM-ZEIT ;  
      measure:BEW ?population ;
      dimension:RAUM ?space;
      dimension:ZEIT ?time .        

    ?obage80 a qb:Observation ;
      qb:dataSet dataset:BEW-RAUM-ZEIT-ALT ;  
      measure:BEW ?age80Plus ;
      dimension:ALT code:ALT9080 ;     
      dimension:RAUM ?space ;
      dimension:ZEIT ?time .   

    ?obage15 a qb:Observation ;
      qb:dataSet dataset:BEW-RAUM-ZEIT-ALT ;  
      measure:BEW ?age15Minus ;
      dimension:ALT code:ALT8015 ;     
      dimension:RAUM ?space ;
      dimension:ZEIT ?time .        

   ?space rdfs:label ?spaceLabel.    

   BIND(SUBSTR(STR(?space),45,4) AS ?spacegroup)         
   FILTER (?spacegroup  IN ('R000', 'R001')) #quarters only
   FILTER(?time = "2017-12-31"^^xsd:date)    
   BIND(STR(?time) AS ?year)  

  }}

GROUP BY ?spaceLabel ?year
ORDER BY ?spaceLabel ?year
```

### ratio of Swiss residents, ratio of short stay residents (Kurzaufenthalter/-innen, mit L-Ausweis), ratio of  'real Zurich people' (Stadtbürger/-innen) pre quarter
```
PREFIX qb: <http://purl.org/linked-data/cube#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dataset: <https://ld.stadt-zuerich.ch/statistics/dataset/>
PREFIX measure: <https://ld.stadt-zuerich.ch/statistics/measure/>
PREFIX dimension: <https://ld.stadt-zuerich.ch/statistics/property/>
PREFIX code: <https://ld.stadt-zuerich.ch/statistics/code/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?spaceLabel ?year
	(SUM(?swiss/?population *100) AS ?swissRatio)
	(SUM(?shortstay/?population *100) AS ?shortstayRatio)
	(SUM(?zurich/?population *100) AS ?zurichRatio)

WHERE{
  GRAPH <https://linked.opendata.swiss/graph/zh/statistics>{   

   ?obpop a qb:Observation ;
      qb:dataSet dataset:BEW-RAUM-ZEIT ;  
      measure:BEW ?population ;
      dimension:RAUM ?space;
      dimension:ZEIT ?time .  

    ?obswiss a qb:Observation ;
      qb:dataSet dataset:BEW-RAUM-ZEIT-HEL ;  
      measure:BEW ?swiss ;
      dimension:HEL code:HEL1000 ;     
      dimension:RAUM ?space;
      dimension:ZEIT ?time .  

    ?obshort a qb:Observation ;
      qb:dataSet dataset:BEW-RAUM-ZEIT-AUA-HEL ;  
      measure:BEW ?shortstay ;
      dimension:HEL code:HEL2000 ;  
      dimension:AUA code:AUA2500 ;         
      dimension:RAUM ?space;
      dimension:ZEIT ?time .        

    ?obzuerich a qb:Observation ;
      qb:dataSet dataset:BEW-RAUM-ZEIT-AUA-HEL ;  
      measure:BEW ?zurich ;
      dimension:HEL code:HEL1000 ;  
      dimension:AUA code:AUA1100 ;         
      dimension:RAUM ?space;
      dimension:ZEIT ?time .    

   ?space rdfs:label ?spaceLabel.

   BIND(SUBSTR(STR(?space),45,4) AS ?spacegroup)  
   FILTER (?spacegroup  IN ('R000', 'R001')) #quarters only
   FILTER(?time = "2017-12-31"^^xsd:date)
   BIND(STR(?time) AS ?year)

  }}

GROUP BY ?spaceLabel ?year
ORDER BY ?spaceLabel ?year
```

### Per quarter: people with short term residents (Ausweis L)
```
PREFIX qb: <http://purl.org/linked-data/cube#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dataset: <https://ld.stadt-zuerich.ch/statistics/dataset/>
PREFIX measure: <https://ld.stadt-zuerich.ch/statistics/measure/>
PREFIX dimension: <https://ld.stadt-zuerich.ch/statistics/property/>
PREFIX code: <https://ld.stadt-zuerich.ch/statistics/code/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?spaceLabel ?year  (SUM(?short/?population *100) AS ?shortStayL)
WHERE{
  GRAPH <https://linked.opendata.swiss/graph/zh/statistics>{   

   ?obpop a qb:Observation ;
      qb:dataSet dataset:BEW-RAUM-ZEIT ;  
      measure:BEW ?population ;
      dimension:RAUM ?space;
      dimension:ZEIT ?time .  

    ?obshort a qb:Observation ;
      qb:dataSet dataset:BEW-RAUM-ZEIT-AUA-HEL ;  
      measure:BEW ?short ;
      dimension:HEL code:HEL2000 ;  
      dimension:AUA code:AUA2500 ;         
      dimension:RAUM ?space;
      dimension:ZEIT ?time .    

   ?space rdfs:label ?spaceLabel.

   BIND(SUBSTR(STR(?space),45,4) AS ?spacegroup)  
   FILTER (?spacegroup  IN ('R000', 'R001')) #quarters only
   FILTER(?time = "2017-12-31"^^xsd:date)
   BIND(STR(?time) AS ?year)

  }}

GROUP BY ?spaceLabel ?year
ORDER BY ?spaceLabel ?year
```

### Swiss ratio per quarter
```
PREFIX qb: <http://purl.org/linked-data/cube#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dataset: <https://ld.stadt-zuerich.ch/statistics/dataset/>
PREFIX measure: <https://ld.stadt-zuerich.ch/statistics/measure/>
PREFIX dimension: <https://ld.stadt-zuerich.ch/statistics/property/>
PREFIX code: <https://ld.stadt-zuerich.ch/statistics/code/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT ?spaceLabel ?year  (SUM(?swiss/?population *100) AS ?swissratio)

WHERE{
  GRAPH <https://linked.opendata.swiss/graph/zh/statistics>{   

   ?obpop a qb:Observation ;
      qb:dataSet dataset:BEW-RAUM-ZEIT ;  
      measure:BEW ?population ;
      dimension:RAUM ?space;
      dimension:ZEIT ?time .    

    ?obswiss a qb:Observation ;
      qb:dataSet dataset:BEW-RAUM-ZEIT-HEL ;  
      measure:BEW ?swiss ;
      dimension:HEL code:HEL1000 ;     
      dimension:RAUM ?space;
      dimension:ZEIT ?time .    

   ?space rdfs:label ?spaceLabel.

   BIND(SUBSTR(STR(?space),45,4) AS ?spacegroup)  
   FILTER (?spacegroup  IN ('R000', 'R001')) #quarters only
   FILTER(?time = "2017-12-31"^^xsd:date)
   BIND(STR(?time) AS ?year)


  }}

GROUP BY ?spaceLabel ?year
ORDER BY ?spaceLabel ?year
```
