# Couch Design Sync

Synchronise CouchDB design documents to always ensure the latest version is in use.

NOTE: This is only in the documentation phase! Nothing real to see here yet!

## Introduction

Ensuring your CouchDB database's design documents are always up to date can quite often be challenging. This
tool aims to simplify to process by synchronising a directory containing a set of design document
definitions.

In addition to ensuring all the design docs exist on each of the databases defined, it will also
attempt to intelligently "migrate" designs so that you don't accidentally upload a new design to a
production database without the views having first been indexed.

In CouchDB, directly replacing a view will cause re-indexation of the data potentially locking usage
of the views until indexation has been completed. This is potentially disastrous on production
system with many gigabytes of data to go through.

When a new design document definition is found, the new document will be created on the database
and a blocking request for the first result of design's first view results will be made. Once
indexation has completed, the process will exit and the design will be primed and ready to use.

If Couch Design Sync detects a design already exists, it will compare the current design document
definition using a checksum calculation. If the checksums don't match, a new design document will
created containing the postfix "_migration" before starting indexation. Once complete, the COPY
action will sent to replace the original design, making the views active.

Much of the logic for Couch Design Sync has been taken from the Ruby CouchRest Model project, and
rewritten in Go for execution convenience.

## Installation

Assuming you have a go environment installed, run:

    go install github.com/samlown/couch-design-sync

Precompiled binary releases will hopefully be made available soon.

## Configuration

The directory from which `couch-design-sync` is called may contain a `couch-design-sync.yml`
configuration file. This tells the command how to connect to the CouchDB server.

If no configuration is found, we'll try to connect to a CouchDB server running on localhost, on the
standard 5984 port and assume it's running under "admin party" mode.

A default configuration could look like:

```yaml
connection:
  protocol: "http"
  host: "localhost"
  port: "5984"
  username: ""
  password: ""
  db_prefix: ""
  db_suffix: ""
```

Note the presence of the db_prefix and db_suffix fields. These will be added to each of the database
names found in the definitions, especially useful for differentiating between deployment
environments.

## Definitions

`couch-design-sync` will by default look in the `./designs` directory for all the information it
needs to synchronise with the server. Inside the design directory, each sub-directory corresponds
to a namespace of design documents. Unless configured otherwise, a namespace corresponds directly
to a database.

Inside each namespace directory we have a list of design document definitions each with either
the `.yml`, `.yaml`, or `.json` extensions.

For example:

```
app
  |-- designs
    |-- notes
      |-- NotesV1.yaml
    |-- users
      |-- Active.yml
      |-- Pending_V2.json
```
