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

## Implementation Options

## Components Detail

## Estimates
