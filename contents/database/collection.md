---
title: "Collection"
description: "Info about how to work with the data collection in Supranim."
keywords: ["data collection", "ozark", "database collection", "orm"]
---

## What is Collection?
Collection is a wrapper around the resulted rows from a query, providing some runtime helper functions, such as sorting,
filtering and mapping operations at runtime, post query execution.

This is useful for performing some operations on the resulted data that are not efficient to
be performed in the database, such as sorting a large dataset, or complex filtering operations that are not supported by the database.

