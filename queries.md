These are the queries included in the paper as well as additional temporalised queries.
They will be included in the git repository that is referenced by the paper to make it easy for people to access an array of queries.

# Competency Question 1

What state type(s) does a `WashingMachine` need to be in for it to be able to perform its primary function.

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX bfo:<http://purl.obolibrary.org/obo/>
PREFIX stasis:<http://www.semanticweb.org/stasis-ontology-filter#>
SELECT ?maintainable_item ?state ?primary_function
WHERE {
  VALUES ?maintainable_item {stasis:washing_machine_001} .
  ?maintainable_item bfo:BFO_0000196 ?primary_function. # BFO_0000196 = "bearer of"
  ?primary_function a stasis:PrimaryFunction .
  { ?state rdfs:subClassOf stasis:Tier1StasisValuePartition } UNION { ?state rdfs:subClassOf stasis:Tier2StasisValuePartition } .
  FILTER NOT EXISTS { ?primary_function stasis:typeDisabledBy ?state }
}
```

The expected result is `On State`, `Operating State`, and `Degraded State` (since it can still function when degraded). The additional anonymous class expressions can be ignored.

Additionally, what state type(s) does a `WashingMachine` need to have for it to be able to realize its capabilities.

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX bfo:<http://purl.obolibrary.org/obo/>
PREFIX core:<http://www.industrialontologies.org/core/>
PREFIX stasis:<http://www.semanticweb.org/stasis-ontology-filter#>
SELECT ?maintainable_item ?state ?capability
WHERE {
  VALUES ?maintainable_item {stasis:washing_machine_001} .
  ?maintainable_item bfo:BFO_0000196 ?capability. # BFO_0000196 = "bearer of"
  ?capability a core:Capability .
  { ?state rdfs:subClassOf stasis:Tier1StasisValuePartition } UNION { ?state rdfs:subClassOf stasis:Tier2StasisValuePartition } .
  FILTER NOT EXISTS { ?capability stasis:typeDisabledBy ?state }
}
```

The expected output is (operate at 5-star efficiency, {on state, operating state}), (turn on, {off state, operating state, degraded state}), (turn off, {on state, operating state, degraded state}).

# Competency Question 2

Can a `WashingMachine` perform its primary function if it is in the `Operating State` but is switched off (i.e., in its `Off State`). Generically, is the `Primary Function` of a `Maintainable Item` disabled in its current `Maintenance State`.

If the query below returns a result for the asset then asset in question is functional, if not then the asset is non-functional.

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX bfo:<http://purl.obolibrary.org/obo/>
PREFIX core:<http://www.industrialontologies.org/core/>
PREFIX stasis:<http://www.semanticweb.org/stasis-ontology-filter#>
SELECT ?maintainable_item ?primary_function
WHERE {
  VALUES ?maintainable_item {stasis:washing_machine_001 stasis:washing_machine_002 stasis:washing_machine_003} .
  ?maintainable_item bfo:BFO_0000196 ?primary_function . # BFO_0000196 = "bearer of"
  ?primary_function a stasis:PrimaryFunction .
  FILTER NOT EXISTS { ?primary_function stasis:disabledBy ?state }
}
```

Expected result is a row for `washing_machine_003` only.

Alternatively, we could retrieve a summary of the functional and non-functional assets with the query below.

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX bfo:<http://purl.obolibrary.org/obo/>
PREFIX core:<http://www.industrialontologies.org/core/>
PREFIX stasis:<http://www.semanticweb.org/stasis-ontology-filter#>
SELECT DISTINCT ?maintainable_item ?primary_function ( !BOUND(?state) AS ?functional )
WHERE {
  VALUES ?maintainable_item {stasis:washing_machine_001 stasis:washing_machine_002 stasis:washing_machine_003} .
  ?maintainable_item bfo:BFO_0000196 ?primary_function . # BFO_0000196 = "bearer of"
  ?primary_function a stasis:PrimaryFunction .
  OPTIONAL { ?primary_function stasis:disabledBy ?state }
}
```

The expected results is (`washing_machine_001`, `wash_clothes`, `false`), (`washing_machine_002`, `wash_clothes`, `false`), (`washing_machine_003`, `wash_clothes`, `true`).

# Competency Question 3

If the asset can perform its primary function, but is not operating at full efficiency, what conditions have not been met? I.e., what `Maintenance State(s)` does the asset need to be in to be able to `realize` the `Capability`.

For this we ensure that the asset is participating in, and is able to participate in, the process of its primary function then identifying the states in which the asset needs to be.

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX bfo:<http://purl.obolibrary.org/obo/>
PREFIX core:<http://www.industrialontologies.org/core/>
PREFIX stasis:<http://www.semanticweb.org/stasis-ontology-filter#>
SELECT ?item ?functioning_process ?capability ( GROUP_CONCAT(?state) as ?states )
WHERE {
  VALUES ?item {stasis:washing_machine_001 stasis:washing_machine_002 stasis:washing_machine_003}
  ?functioning_process a stasis:FunctioningProcess .
  ?item bfo:BFO_0000056 ?functioning_process  # 'participates in at some time'
  FILTER NOT EXISTS { ?item stasis:unableToParticipateIn ?functioning_process } .
  ?item bfo:BFO_0000196 ?capability . # bearer of
  ?capability a stasis:OperateAtFiveStarEfficiency .
  OPTIONAL { ?capability bfo:BFO_0000054 ?process } FILTER ( !BOUND(?process) ) .
  { ?state rdfs:subClassOf stasis:Tier1StasisValuePartition } UNION { ?state rdfs:subClassOf stasis:Tier2StasisValuePartition } 
  MINUS {
    {?capability stasis:typeDisabledBy ?state} UNION {?item stasis:hasState/a ?state}
  } 
}
GROUP BY ?item ?functioning_process ?capability
```

The expected result is (`washing_machine_003`, `wash_clothes_process_C`, {`operating state`})

