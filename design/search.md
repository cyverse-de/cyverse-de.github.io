# Search Redesign

## Motivation

Our existing Elasticsearch-based search system has several issues:

- Elasticsearch adds operational complexity to our system that we haven't been keeping up with
- Elasticsearch being used for only data search means our search capabilities and syntax vary between different sorts of search, which additionally makes a "universal search" as planned for Sonora hard to plan and build
- We aren't well-placed to take advantage of many advanced features of Elasticsearch, and our amount of data being searched is relatively small

## Architecture Changes

At a high level:

- All search will move to using a PostgreSQL-based system with a single, separate database from our existing source database used to store searchable information
- This PostgreSQL database will use well-defined relational tables, moving our search away from a JSON document-based system. Indexes will be determined by what sorts of matching are enabled by the services querying the database.
- Current indexing tools will be replaced by a set of tools built on the same basic model: a "fully index once" mode, a periodic reindexing mode, and an incremental mode.
- Current search services and endpoints will be replaced by a service or set of services with a consistent interface mapping queries expressed in a custom JSON format to underlying PostgreSQL queries and formatting the output as JSON for consumption
- Where possible, this custom JSON query language will match the existing data search query language
- An additional endpoint or service will be created that aggregates results across several searchable entities, integrating sensible defaults to enable a universal search as well as providing a familiar query-string type search for use by frontend tools

Given the sweeping nature of the changes, a few other things will naturally happen that aid consistency and reusability:
- Natural language strings should always be marked with a language, even if it's always English for now
- Dates should be stored as a point in time, with a separate field used if the modifying user's timezone or locale is relevant

## Components Overview
### Database

The database will have a simplified, denormalized version of the underlying data, somewhat similar to the existing Elasticsearch index structure. For example, data would most likely operate off four tables: data (with all the base information), metadata (with both iRODs and DE metadata), permissions, and tags. Apps and analyses would be structured similarly -- a base table with anything of a simple structure (no matter how deeply stored in our other databases), plus extra tables for any sort of variable-sized sub-collections. One such type of sub-collection might be for natural-language strings, which should be stored along localization information, as noted above.

### Indexing tools

These tools will look a lot like our existing tools, except normalized and, of course, using database queries rather than Elasticsearch ones. The existing service these will mimic, where possible, is templeton: have one tool that operates in several modes, with the different modes settting up different ways of triggering the same underlying code paths. Both incremental and periodic modes will be controlled by AMQP messages, as they currently are. One place where this one-service model may not be possible or preferred is for data indexing, where we may wish to retain the split between a Jargon-based tool for incremental indexing (dewey) and a faster, direct database tool for periodic indexing (infosquito). It may be necessary, particularly with iRODs, to create intermediate services that translate messages coming out of that system into a more consumable format for ours, for example deduplicating many subsequent messages affecting the same data, or splitting messages that affect many sections of data.

### Search services

These tools will look a lot like the existing search service (for data search), and may even grow out of it. Their basic structure will be as translators: take in a query, translate it as needed to an underlying query or queries, get results, and translate them into a useful output format. We'll need to create new tools to translate queries into SQL rather than elasticsearch, and probably better tools for assembling documents to return, since Elasticsearch did that for us before. The one entirely new feature would be a universal-search/aggregator implementation.

## Implementation Options

There's not a whole lot of choices to be made here, really. Other than continuing to use Elasticsearch, Solr, some sort of other custom solution (maybe using Xapian or similar), etc., which would therefore not use, or at least not significantly use, PostgreSQL, the options lie within the capabilities of the DBMS. Perhaps the only significant choice is exactly what fields are indexed and in what ways within the database system, and accordingly how queries are constructed to take advantage of it. According to my research, there's basically three relevant types of indices we can use: regular b-tree indices, which are useful for simple comparisons like equality, less-than, greater-than, and things that can be reduced to them (such as LIKE and ILIKE with no leading wildcard) -- these are useful to some degree on almost every field; full-text-search GIN indices, which are useful for natural-language searching with associated tokenization, stemming, etc -- these are useful only for a select few fields that can be searched as natural language strings; and trigram GIN indices (made possible by the pg_trgm extension), which improve the speed of LIKE, ILIKE, and regular-expression searches that can be decomposed into trigrams -- these are most useful for fields that use these operators.

Accordingly, the indexing strategy will in general be: b-tree indices on almost everything, trigram indices on text fields where wildcarded search terms are valuable, and full-text-search indices on text representing natural language text (which as discussed earlier will be marked with locale information, additionally useful for search).

## Components Detail

### Database schemata

#### Data

The main data table:

 - id uuid not null
 - path text not null
 - label varchar(1000) not null
 - creator text not null
 - datecreated timestamp with time zone not null
 - datemodified timestamp with time zone not null
 - filesize bigint
 - filetype varchar(250)

Metadata:

 - id uuid (used for DE metadata that has IDs)
 - type text not null
 - attribute text not null
 - value text not null
 - unit text
 - datecreated timestamp with time zone not null
 - datemodified timestamp with time zone not null

## Estimates
