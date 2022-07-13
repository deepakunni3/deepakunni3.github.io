---
title: "Biolink Model"
description: "A high level data model for organizing biological and biomedical knowledge"
summary: "A high level data model for organizing biological and biomedical knowledge."
date: 2022-07-13
showTableOfContents: true
tags: ["data modeling"]
---

In this blog post, I'll talk about Knowledge Graphs (KGs), some of the challenges while working with KGs (especially biological or biomedical KGs), and then introduce Biolink Model as a way of tackling the challenges described. Towards the end, I'll also provide a few examples on how to use the Biolink Model to structure knowledge in the biological and biomedical domain.


## Knowledge Graphs

There are a handful of definitions for KGs but the one that I have found to be the most succinct is:

A Knowledge Graph is a graph that represents knowledge where entities are represented as nodes and relationship between these entities are represented as edges. This graph may conform to a graph-based data model which maybe either a property graph or a directed edge-labelled graph.

There are several KGs out there in different domains, ranging from commerical to research KGs. More recently, there has been a proliferation of KGs that are significance to the biological, biomedical, and clinical domains. With this proliferation there is a rising complexity of how these graphs are represented, the technologies used, the modeling decisions made, and the usability of graphs outside the context for which they were originally intended for.

## Challenges with KGs

KGs built by different research groups, projects, and initiatives are largely incompatible due to lack of unifying standards for knowledge representation. While there are several vocabularies, data models, and ontologies for representing nodes and edges in a graph, there is a lack of a briding data model that unifies related concepts across vocabularies, terminologies, and ontologies.

Some of the mostly common encountered challenges are described below:

**Graph Formalism:** The graph formalism used for representing nodes and edges differ between KGs. For example, one KG might use a property graph model where information like provenance, evidence, and any additional metadata are attached directly to the edges. While another KG might adopt the RDF formalism, which typically doesn't allow for edge properties. There are ways around that where one can use [RDF Reification](https://www.w3.org/wiki/RdfReification) or [RDF-Star](https://w3c.github.io/rdf-star/cg-spec/editors_draft.html). But all of this means that the representation is still KG specific and not entirely clear to an external user/collaborator/data engineer.

**Vocabulary:** The vocabulary used to type nodes and edges differ between KGs. These differences can be either syntactic or semantic.

- **Syntactic:** The same concept being represented in a lexically different manner. For example, ‘Gene’ vs ‘gene’.
- **Semantic:** The same concept represented at different levels of granularity. For example, ‘Gene’ vs ‘genomic feature’, where both are correct but the former represents a more specific semantic type.

**Schema**: A schema, or lack thereof, can often lead to difficulties when working with multiple heterogeneous KGs. Some KGs, especially property graphs, do not have a schema. And if they do, these may or may not be machine readable which makes it difficult to understand ahead of time what the structure of a KG would be and the vocabulary used. There is also a lack of agreement on what the schema would look like. For example, is the schema represented as an ontology OWL, SQL schema, JSON Schema, or in some proprietary form.

**Identifiers**: KGs typically ingest data from several sources. One KG might ingest a source in a particular way while another KG might use a slightly different (but incompatible) approach. An example of such variation is in the choice of identifiers used to represent nodes. For example, let's take the example of nodes that represent genes. Certain KGs may use identifiers from `NCBIGene` namespace to refer to a gene while another might use `HGNC`, and yet another KG might use a mix of the two.

**Modeling**: KGs are graph representations of data from various sources and enriched with semantics. In many cases data from these sources are not essentially in a graph form. Thus, there is a need for a data ETL (Extract-Transform-Load) pipeline that would transform the data sources and map the knowledge onto an implicit graph model. Different groups may have different approaches on how to interpret and transform these sources.


All of these challenges makes it difficult to reuse KGs outside the context for which they were originally designed for and thus getting in the way of interoperability and reusability.

There is a need for a common data model that is flexible, covers a wide range of domains, and acts as a bridge between various vocabularies, terminologies and ontologies.


## Biolink Model

The Biolink Model is a high-level data model of biological entities and the associations between these entities. Biolink Model provides a common dialect for representing knowledge when building biological and biomedical KGs. The model was designed as a way of standardizing types and relational structures in KGs while being agnostic to the graph formalism used for knowledge representation. The model is built using [LinkML](https://linkml.io/), a linked-data modeling framework, and exists as a YAML. This YAML, the source of truth for the model, is then used to generate various technology-specific artifacts like JSONSchema, Python dataclasses, Java classes, RDF, OWL, and RDF Shapes. This diversity in artifacts allow for Biolink Model to be used in various downstream technology stacks, depending on the intended application.


{{< mermaid >}}

graph TD;

linkml:Definition-->linkml:ClassDefinition;
linkml:Definition-->linkml:SlotDefinition;

linkml:ClassDefinition-->biolink:Entity;
biolink:Entity-->biolink:NamedThing;
biolink:Entity-->biolink:Association;
linkml:SlotDefinition-->biolink:node_property;
linkml:SlotDefinition-->biolink:association_slot;
linkml:SlotDefinition-->biolink:related_to;


click linkml:Definition "https://w3id.org/linkml/Definition" _blank
click linkml:ClassDefinition "https://w3id.org/linkml/ClassDefinition" _blank
click linkml:SlotDefinition "https://w3id.org/linkml/SlotDefinition" _blank

click biolink:Entity "https://w3id.org/biolink/vocab/Entity" _blank
click biolink:NamedThing "https://w3id.org/biolink/vocab/NamedThing" _blank
click biolink:Association "https://w3id.org/biolink/vocab/Association" _blank
click biolink:node_property "https://w3id.org/biolink/vocab/node_property" _blank
click biolink:association_slot "https://w3id.org/biolink/vocab/association_slot" _blank
click biolink:related_to "https://w3id.org/biolink/vocab/related_to" _blank


classDef textBox font-size: 90%
class linkml:Definition,linkml:ClassDefinition,linkml:SlotDefinition textBox
class biolink:Entity,biolink:NamedThing,biolink:Association textBox
class biolink:node_property,biolink:association_slot,biolink:related_to textBox

classDef linkmlBox fill: white
class linkml:Definition,linkml:ClassDefinition,linkml:SlotDefinition linkmlBox

{{< /mermaid >}}

The Biolink Model consists of 5 main parts:
- [Classes](#classes)
- [Associations](#associations)
- [Predicates](#predicates)
- [Node Properties](#node-properties)
- [Edge Properties](#edge-properties)

Each of the above are defined using [LinkML vocabulary](https://linkml.io/linkml-model/docs/) and collectively represents the major part of the model.


#### Classes

Classes are high-level types that represent core concepts such as genes, proteins, diseases, phenotypic features, and chemical substances. These classes are arranged in a hierarchy based on their semantics.

For example,

```yaml
  phenotypic feature:
    aliases: ['sign', 'symptom', 'phenotype', 'trait', 'endophenotype']
    is_a: disease or phenotypic feature
    description: >-
      A combination of entity and quality that makes 
      up a phenotyping statement.
    exact_mappings:
      - UPHENO:0001001
      - SIO:010056
      - WIKIDATA:Q104053
    narrow_mappings:
      - STY:T184
      - WIKIDATA:Q169872
      - WIKIDATA:Q25203551
    id_prefixes:
      - HP
      - EFO
      - NCIT
      - UMLS
      - MEDDRA
      - MP
      - ZP
      - UPHENO
      ...
```

The above is a truncated example of the definition for `phenotypic feature` in Biolink Model 2.4.4

Each class has,
- *aliases:* Alternate names for the same class
- *description:* A human readable description. Similar to `rdfs:label`
- *mappings:* Mappings - to other vocabularies, terminologies, and ontologies - organized according to the level of granularity of mappings
- *identifier prefixes* - A list of preferred prefixes for identifiers used to represent instances of this class

#### Associations

Associations are classes that represent a statement (or an assertion), where the statement defines a relationship between two concepts, along with any additional information that provides more context to the nature of the statement.

In RDF sense,

The statement (or assertion) is a triple that contains a subject, a predicate, and an object.

{{< mermaid >}}

graph LR
    A(subject) -->|predicate| B(object)

{{< /mermaid >}}


The Association is a reified statement that has additional information attached to the statement.

{{< mermaid >}}

flowchart LR
    subgraph biolink:Association
        S(subject)
        P(predicate)
        O(object)
    end;
biolink:Association-->|biolink:has_publication| P1(P1)
biolink:Association-->|biolink:has_evidence| E1(E1)

{{< /mermaid >}}


#### Predicates

Predicates are relationships that can be used to link two nodes together in an association.

For example,

```yaml
  has phenotype:
    is_a: related to at instance level
    aliases: ['disease presents symptom']
    description: >-
      holds between a biological entity and a phenotype, where a phenotype
      is construed broadly as any kind of quality of an organism part,
      a collection of these qualities, or a change in quality or qualities
      (e.g. abnormally increased temperature). 
      In SNOMEDCT, disorders with keyword 'characterized by' should
      translate into this predicate.
    domain: biological entity
    range: phenotypic feature
    multivalued: true
    exact_mappings:
      - RO:0002200
    broad_mappings:
      - NCIT:R115
      - NCIT:R108
    narrow_mappings:
      - NCIT:R89
      - DOID-PROPERTY:has_symptom
      - RO:0004022
    ...
```

The above is a truncated example of the definition for `has phenotype` in Biolink Model 2.4.4

Each predicate has,
- *aliases:* Alternate names for the same slot
- *description:* A human readable description. Similar to `rdfs:label`
- *domain*: The type of class that is the subject when this predicate is used in a triple. Similar to `rdfs:domain`
- *range*: The type of class that is the object when this predicate is used in a triple. Similar to `rdfs:range`
- *multivalued*: Whether the range of this slot are multivalued
- *mappings:* Mappings - to other vocabularies, terminologies, and ontologies - organized according to the level of granularity of mappings. Typically predicates are mapped to [Relations Ontology](https://obofoundry.org/ontology/ro.html)


#### Node Properties

Node properties are properties that can be used to describe nodes in a graph.

```yaml
  name:
    aliases: [ 'label', 'display name', 'title' ]
    description: >-
      A human-readable name for an attribute or entity.
    range: label type
    in_subset:
      - translator_minimal
      - samples
```

#### Edge Properties

Edge properties are properties that can be used to describe an association in a graph.

```yaml
  has evidence:
    is_a: association slot
    range: evidence type
    description: >-
      connects an association to an instance of supporting evidence
    exact_mappings:
      - RO:0002558
    multivalued: true
```


## Examples

To show the Biolink Model in practice, following is an example on how a small graph (3 nodes and 1 edge) could be represented using Biolink Model.

{{< mermaid >}}

flowchart LR
    subgraph biolink:DiseaseToPhenotypicFeatureAssociation
        S(MONDO:0005027)
        P(biolink:has_phenotype)
        O(HP:0001272)
    end;
biolink:DiseaseToPhenotypicFeatureAssociation-->|biolink:has_publication| P1(PMID:17220172)
{{< /mermaid >}}

The graph is representing an association, of type `biolink:DiseaseToPhenotypicFeatureAssociation`, between `MONDO:0005027 epilepsy` and `HP:0001272 Cerebellar atrophy` with the predicate `biolink:has_phenotype`. The publication, `PMID:17220172`, based on which this association was made is represented using the `biolink:has_publication` property.


Now, there are more than one way to serialize a graph into a form that can be exchanged and archived.

The most common (and easily readable) serialization forms are:
- [JSON](#json)
- [RDF Turtle](#rdf-turtle)
- [RDF-Star](#rdf-star)


#### JSON

JSON is an easy way to share data and same holds true for sharing graphs. And there is no requirement for complicated parsers for reading graphs in JSON.

Below you can see the example graph serialized into JSON.

```json
{
    "nodes":
    [
        {
            "biolink:id": "MONDO:0005027",
            "biolink:name": "epilepsy",
            "biolink:category": [ "biolink:Disease" ]
        },
        {
            "biolink:id": "HP:0001272",
            "biolink:name": "Cerebellar atrophy",
            "biolink:category": [ "biolink:PhenotypicFeature" ]
        },
        {
            "biolink:id": "PMID:17220172",
            "biolink:category": [ "biolink:Publication" ]
        }
    ],
    "edges":
    [
        {
            "biolink:subject": "MONDO:0005027",
            "biolink:predicate": "biolink:has_phenotype",
            "biolink:object": "HP:0001272",
            "biolink:category": [
            	"biolink:DiseaseToPhenotypicFeatureAssociation"
        	],
            "biolink:has_publication": [ "PMID:17220172" ]
        }
    ]
}
```

#### RDF Turtle

RDF Turtle is one of the serialization formats for RDF graphs. 

Below you can see the example graph serialized into RDF Turtle.

```rdf
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix biolink: <https://w3id.org/biolink/vocab/> .
@prefix MONDO: <http://purl.obolibrary.org/obo/MONDO_> .
@prefix HP: <http://purl.obolibrary.org/obo/HP_> .
@prefix PMID: <http://www.ncbi.nlm.nih.gov/pubmed/> .
@prefix ex: <www.example.com/> .

MONDO:0005027 a biolink:Disease ;
    rdfs:label "epilepsy" .

HP:0001272 a biolink:PhenotypicFeature ;
    rdfs:label "Cerebellar atrophy" .

PMID:17220172 a biolink:Publication .


ex:association1 a biolink:DiseaseToPhenotypicFeatureAssociation ;
    rdf:subject MONDO:0005027 ;
    rdf:predicate biolink:has_phenotype ;
    rdf:object HP:0001272 ;
    biolink:has_publication PMID:17220172 . 
```


#### RDF-Star

RDF-Star is an extension to RDF for representing properties on edges without having to use RDF Reification.

Feel free to refer to the [W3C RDF-Star specification](https://w3c.github.io/rdf-star/cg-spec/2021-12-17.html) for more information.

Below you can see the example graph serialized into RDF Turtle where associations are represented using RDF-Star.

```rdf
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix biolink: <https://w3id.org/biolink/vocab/> .
@prefix MONDO: <http://purl.obolibrary.org/obo/MONDO_> .
@prefix HP: <http://purl.obolibrary.org/obo/HP_> .
@prefix PMID: <http://www.ncbi.nlm.nih.gov/pubmed/> .

MONDO:0005027 a biolink:Disease ;
    rdfs:label "epilepsy" .

HP:0001272 a biolink:PhenotypicFeature ;
    rdfs:label "Cerebellar atrophy" .

PMID:17220172 a biolink:Publication .


<MONDO:0005027 biolink:has_phenotype HP:0001272> a biolink:DiseaseToPhenotypicFeatureAssociation ;
    biolink:has_publication PMID:17220172 .

```

> **Note:** You do need a triple store that is capable of interpreting RDF-Star to be able to use the above example in a meaningfuly way.


## Future Directions

The Biolink Model is a data model for organizing and representing biological and biomedical KGs. It tackles all of the aforementioned challenges by creating a common dialect for building and exchanging KGs.

The Biolink Model is curated by a growing community and is primarily funded by the [NCATS Biomedical Data Translator](https://ncats.nih.gov/translator/about) program. The model is built in an iterative manner depending on use-cases and requirements. The Biolink Model has seen adoption in various other initiatives such as the Monarch Initiative, Illuminating the Druggable Genome Knowledge Graph (KG-IDG), KG-COVID-19, and KG-Microbe.

To know more about the model, read the [latest paper](https://ascpt.onlinelibrary.wiley.com/doi/10.1111/cts.13302) and visit [Biolink Model](https://github.com/biolink/biolink-model/) on GitHub {{< icon "github" >}}.

