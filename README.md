# OpenFGA Skill for Claude Code

A Claude Code skill for testing and managing [OpenFGA](https://openfga.dev) authorization stores — all through natural language, all via the `fga` CLI.

## What it does

Talk to your OpenFGA instance in plain English. The skill translates your intent into `fga` CLI commands and shows you exactly what it ran.

**Manage stores** — list, create, delete, import, export
**Manage models** — write, list, validate, test
**Manage tuples** — write (inline or from file), read, delete
**Run queries** — check, list-objects, list-users, expand
**Run test suites** — from a `.yaml` test file

## Setup

Requires the [fga CLI](https://github.com/openfga/cli) installed and an OpenFGA instance running.

On first use, run:
```
/openfga connect
```

The skill will ask for your OpenFGA URL and API token, verify the connection, and save them to `.fga-skill.env` in your working directory.

## Usage

```
/openfga <operation> [args]
```

---

## Example conversations

---

### Check if a user can read an organization

```
> Can alice read org_1?

  Running an authorization check for user:alice on organizations_read
  for organization:org_1.

  $ fga query check \
      --store-id 01KB2ACG5DCJ1C5GQSRZV1SF7M \
      --api-url http://localhost:9080 \
      --api-token $FGA_TOKEN \
      --config /dev/null \
      user:alice organizations_read organization:org_1

  {"allowed": true, "resolution": ""}

  ✅ ALLOWED — user:alice has organizations_read on organization:org_1
```

---

### Add a relationship tuple

```
> Add alice as a member of org_1 via membership:mem_alice

  Writing the membership tuple for alice.

  $ fga tuple write \
      --store-id 01KB2ACG5DCJ1C5GQSRZV1SF7M \
      --api-url http://localhost:9080 \
      --api-token $FGA_TOKEN \
      --config /dev/null \
      user:alice user membership:mem_alice

  {"successful_count": 1, "failed_count": 0}

  Tuple written. user:alice is now the user of membership:mem_alice.
```

---

### Load a model from a file

```
> Write the model from v1.fga

  Uploading the authorization model from v1.fga.

  $ fga model write \
      --file v1.fga \
      --store-id 01KB2ACG5DCJ1C5GQSRZV1SF7M \
      --api-url http://localhost:9080 \
      --api-token $FGA_TOKEN \
      --config /dev/null

  {"authorization_model_id": "01KJMTACB841CW51EXE2H0997Z"}

  New model ID: 01KJMTACB841CW51EXE2H0997Z
```

---

### Bulk insert tuples from a test file

```
> Insert all the tuples from test.yaml

  test.yaml is a full test file — extracting the tuples: section
  and writing them.

  $ fga tuple write \
      --file /tmp/fga-tuples-extracted.yaml \
      --store-id 01KB2ACG5DCJ1C5GQSRZV1SF7M \
      --api-url http://localhost:9080 \
      --api-token $FGA_TOKEN \
      --config /dev/null

  42 tuples written successfully (0 failed) in 210ms
```

---

### List all organizations a user can access

```
> What organizations can avinash write to?

  Listing all organization objects where user:avinash has organizations_write.

  $ fga query list-objects \
      --store-id 01KB2ACG5DCJ1C5GQSRZV1SF7M \
      --api-url http://localhost:9080 \
      --api-token $FGA_TOKEN \
      --config /dev/null \
      user:avinash organizations_write organization

  {"objects": ["organization:org_1", "organization:org_2"]}

  user:avinash can write to organization:org_1 and organization:org_2.
```

---

### Run a full test suite

```
> Run all the tests in test.yaml

  Running the full test suite from test.yaml.

  $ fga model test \
      --tests test.yaml \
      --store-id 01KB2ACG5DCJ1C5GQSRZV1SF7M \
      --api-url http://localhost:9080 \
      --api-token $FGA_TOKEN \
      --config /dev/null

  Tests 35/37 passing
  Checks 97/100 passing
  ListObjects 28/28 passing

  (FAILING) Avinash can read env_1: Checks (1/2 passing)
  x Check(user=user:avinash, relation=environment_write, object=environment:env_1):
    expected=false, got=true
```

---

## Operations reference

| What you say | Operation |
|---|---|
| connect / set up connection | `connect` |
| list stores / what stores exist | `list-stores` |
| create a store called X | `create-store <name>` |
| switch to store X | `use-store <id>` |
| write model from file | `write-model <file>` |
| list models / show model history | `list-models` |
| validate the model | `validate-model <file>` |
| add this tuple / write relationship | `write-tuples <user> <relation> <object>` |
| insert tuples from file | `write-tuples <file>` |
| show all tuples / read relationships | `read-tuples` |
| remove this relationship | `delete-tuples <user> <relation> <object>` |
| can X do Y on Z? | `check <user> <relation> <object>` |
| what can X access? | `list-objects <user> <relation> <type>` |
| who can access X? | `list-users <object> <relation> <type>` |
| expand this relation | `expand <relation> <object>` |
| run tests | `run-tests <file>` |
| export the store | `export-store` |
| import store from file | `import-store <file>` |
