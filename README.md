# Analyze Android APK files with jQAssistant 

This is a simple prototype to run jQAssistant tests on an apk file. 


## Introduction
This script decompiles an apk to its corresponding java jar file and scan this file with jQAssistant. 

 ***This script just automates the sequence in which various tools are used and does not handle any error events. You will have to go through the cmd output to figure out the problem.***

## Getting started

### Requirements

* JRE 1.7 (Java Runtime Environment)
* Python 3


### Actions


**Test**

Decompile the apk, analyze the jar, check the [rules](./rules)  and creates an report.

```sh 

$ python apk_to_jqassistant.py test yourFile.apk

```

**Server**

Just starts the neo4J server instance 

```sh 

$ python apk_to_jqassistant.py server

```

**Analyze**

Only check the [rules](./rules) and creates the report 

```sh 

$ python apk_to_jqassistant.py analyze

```


## Architecture 

![Flow](./docs/architecture.png)

## Example output 

This is just a simple example output of a apk that I downloaded form the google play store. 

## Queries

### Metrics

```cypher
OPTIONAL MATCH (t:Type:Class) WITH count(t) as classes
OPTIONAL MATCH (t:Type:Interface) WITH classes, count(t) as interfaces
OPTIONAL MATCH (t:Type:Enum) WITH classes, interfaces, count(t) as enums
OPTIONAL MATCH (t:Type:Annotation) WITH  classes, interfaces, enums, count(t) as annotations
OPTIONAL MATCH (t:Type:Class)-[:DECLARES]->(m:Method) WITH classes, interfaces, enums, annotations, count(m) as methods, sum(1) as loc
OPTIONAL MATCH (t:Type:Class)-[:DECLARES]->(f:Field) RETURN classes, interfaces, enums, annotations, methods,  count(f) as fields

```


### Deprecations

```cypher
match
(t:Type)-[:ANNOTATED_BY]->(:Annotation)-[:OF_TYPE]->(d:Type),
(t)<-[:DEPENDS_ON]-(caller)<-[:CONTAINS*..]-(tLevel2:Package)<-[:CONTAINS]-(p:Package {name: "amazon"})
where
  d.fqn="java.lang.Deprecated"
with "com." + p.name + "." + tLevel2.name as packagePrefix, caller, t
return caller.fqn as caller, t.fqn as callee, count(caller) as numberOfDeprecationUsages

```


### Exceptions

```
match
  (e:Exception)-[:DECLARES]->(c:Constructor),
  (a:Package {name:"amazon"})-[:CONTAINS*..]->(t:Type)-[:DECLARES]->     (m:Method)-[i:INVOKES]->(c)
where
  not (m:Constructor)
  and e.fqn =~ "java\\.lang\\..*"
return
  e.fqn as exception, count(t)
order by
  exception,  type
```


## Dependencies

* [Dex2jar](http://code.google.com/p/dex2jar/)
* [jQAssistant](https://jqassistant.org/)

