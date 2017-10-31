---
layout: page
title: DE API Documentation
---

# Table of Contents

* [Reference Genome endpoints](#reference-genome-endpoints)
    * [Exporting Reference Genomes](#exporting-reference-genomes)
    * [Get a Reference Genome by ID](#get-a-reference-genome-by-id)
    * [Adding Reference Genomes](#adding-reference-genomes)
    * [Deleting Reference Genomes](#deleting-reference-genomes)
    * [Updating Reference Genomes](#updating-reference-genomes)

# Reference Genome endpoints

Note that secured endpoints in Terrain and apps are a little different from each other. Please see [Terrain Vs. Apps](terrain-v-apps.html) for more information.

## Exporting Reference Genomes

Secured Endpoint: GET /reference-genomes

Delegates to apps: GET /reference-genomes

This endpoint is a passthrough to the apps endpoint with the same path. Please see the apps service documentation for more details.

## Get a Reference Genome by ID

Secured Endpoint: GET /reference-genomes/{reference-genome-id}

Delegates to apps: GET /reference-genomes/{reference-genome-id}

This endpoint is a passthrough to the apps endpoint with the same path. Please see the apps service documentation for more details.

## Adding Reference Genomes

Secured Endpoint: POST /admin/reference-genomes

Delegates to apps: POST /admin/reference-genomes

This endpoint is a passthrough to the apps endpoint with the same path. Please see the apps service documentation for more details.

## Deleting Reference Genomes

Secured Endpoint: DELETE /admin/reference-genomes/{reference-genome-id}

Delegates to apps: DELETE /admin/reference-genomes/{reference-genome-id}

This endpoint is a passthrough to the apps endpoint with the same path. Please see the apps service documentation for more details.

## Updating Reference Genomes

Secured Endpoint: PATCH /admin/reference-genomes/{reference-genome-id}

Delegates to apps: PATCH /admin/reference-genomes/{reference-genome-id}

This endpoint is a passthrough to the apps endpoint with the same path. Please see the apps service documentation for more details.
