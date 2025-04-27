---
title: "Changing Api With No Downtime"
date: 2025-04-21T16:59:23+01:00
draft: true
---

## Problem

You are trying to change an existing API and you don't want to have downtime.

## System

React Frontend + Python Backend + Postgres DB

```json
{
  "name": "John Doe",
  "telephone": "123456789",
  "address": {
    "firstLine": "123 Fake Street",
    "postcode": "E1 8DX",
    "fromData": "2020-01-01"
  }
}
```

```json
{
  "name": "John Doe",
  "telephone": "123456789",
  "addresses": [
    {
      "firstLine": "123 Fake Street",
      "postcode": "E1 8DX",
      "fromData": "2020-01-01"
    }
  ]
}
```

### Changes we want to make

1. We're adding a field to the `fromData`
2. We're converting the address from a single object to an array of objects

## Blanking on out ORMs

Take a look at the deep dive here.
