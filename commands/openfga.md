| description | argument-hint | allowed-tools |
|---|---|---|
| Interact with an OpenFGA instance — manage stores, models, tuples, and run authorization checks using the fga CLI | <connect\|list-stores\|use-store\|write-model\|list-models\|get-model\|validate-model\|write-tuples\|read-tuples\|delete-tuples\|check\|list-objects\|list-users\|expand\|run-tests\|export-store\|import-store\|create-store\|delete-store> [args...] | Bash |

# OpenFGA Skill

**Arguments:** $ARGUMENTS

---

## Your task

Parse `$ARGUMENTS` to determine the operation and run the appropriate `fga` CLI command.

---

## Step 1 — Load connection config

Read `.fga-skill.env` from the current working directory. It should contain:

```
FGA_URL=http://localhost:9080
FGA_TOKEN=<api-token>
FGA_STORE_ID=<store-id>
```

Use `cat .fga-skill.env 2>/dev/null` to read it. Export the variables for use in all subsequent commands.

If the file is missing or `FGA_URL` / `FGA_TOKEN` are empty, skip to the **`connect` operation** below before doing anything else.

For operations that require a store (anything except `connect`, `list-stores`, `create-store`): if `FGA_STORE_ID` is empty, run `fga store list` to show available stores and ask the user which one to use, then save the choice.

---

## Step 2 — Always check FGA is reachable before any operation

Run:
```bash
fga store list --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null
```

If this fails, show the error clearly and tell the user to verify their URL and token. Do not proceed.

**Critical flags — include on every fga command:**
- `--api-url $FGA_URL`
- `--api-token $FGA_TOKEN`
- `--config /dev/null`  ← prevents ~/.fga.yaml from overriding with cloud credentials

---

## Step 3 — Execute the operation

### `connect`

Prompt the user for:
- **OpenFGA URL** (e.g. `http://localhost:9080`)
- **API Token**

Then verify by running `fga store list` with those credentials. If successful, list the available stores and ask the user to pick one (or skip store selection if they just want to connect).

Save to `.fga-skill.env`:
```
FGA_URL=<url>
FGA_TOKEN=<token>
FGA_STORE_ID=<chosen-store-id or empty>
```

---

### `list-stores`

```bash
fga store list --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null
```

Display the store names and IDs in a readable table.

---

### `create-store <name>`

```bash
fga store create --name <name> --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null
```

Show the new store ID and ask the user if they want to use it as the active store. If yes, update `FGA_STORE_ID` in `.fga-skill.env`.

---

### `delete-store [store-id]`

Use `store-id` from argument if provided, otherwise use `$FGA_STORE_ID`.

```bash
fga store delete --store-id <id> --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null
```

Confirm with the user before deleting.

---

### `use-store <store-id>`

Update `FGA_STORE_ID` in `.fga-skill.env` to the given store ID. No API call needed — just save the value.

---

### `write-model <file>`

```bash
fga model write --file <file> --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null
```

Print the returned authorization model ID prominently.

---

### `list-models`

```bash
fga model list --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null
```

---

### `get-model [model-id]`

If a model ID is provided:
```bash
fga model get --model-id <id> --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null
```

Without model ID, returns the latest model:
```bash
fga model get --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null
```

---

### `validate-model <file>`

```bash
fga model validate --file <file> --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null
```

---

### `write-tuples <file-or-tuple>`

**If argument is a file path:**

First check if it's a full test YAML (contains a `tuples:` key alongside `model_file:` or `tests:`). If so, extract just the tuples section using Node.js:

```bash
node -e "
const fs = require('fs');
const content = fs.readFileSync('<file>', 'utf8');
const lines = content.split('\n');
let inTuples = false, result = [];
for (const line of lines) {
  if (line.match(/^tuples:/)) { inTuples = true; continue; }
  if (line.match(/^(tests|name|model_file):/)) { inTuples = false; }
  if (inTuples) result.push(line);
}
fs.writeFileSync('/tmp/fga-tuples-extracted.yaml', result.join('\n'));
" && fga tuple write --file /tmp/fga-tuples-extracted.yaml --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null
```

If it's already a plain tuples YAML/JSON/CSV file:
```bash
fga tuple write --file <file> --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null
```

**If argument is an inline tuple** (`<user> <relation> <object>`):
```bash
fga tuple write --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null <user> <relation> <object>
```

---

### `read-tuples [--user <u>] [--relation <r>] [--object <o>]`

```bash
fga tuple read --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null [--user <u>] [--relation <r>] [--object <o>]
```

All filters are optional.

---

### `delete-tuples <file-or-tuple>`

**If argument is a file path:**
```bash
fga tuple delete --file <file> --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null
```

**If argument is an inline tuple** (`<user> <relation> <object>`):
```bash
fga tuple delete --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null <user> <relation> <object>
```

---

### `check <user> <relation> <object>`

```bash
fga query check --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null <user> <relation> <object>
```

Display result clearly: **ALLOWED** or **DENIED**.

---

### `list-objects <user> <relation> <type>`

```bash
fga query list-objects --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null <user> <relation> <type>
```

---

### `list-users <object> <relation> <user-filter-type>`

```bash
fga query list-users --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null <object> <relation> <user-filter-type>
```

---

### `expand <relation> <object>`

```bash
fga query expand --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null <relation> <object>
```

---

### `run-tests <file>`

```bash
fga model test --tests <file> --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null
```

Show a summary of passing/failing tests. Highlight any failures with the expected vs actual values.

---

### `export-store [--output <file>]`

```bash
fga store export --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null [--output-file <file>]
```

---

### `import-store <file>`

```bash
fga store import --file <file> --store-id $FGA_STORE_ID --api-url $FGA_URL --api-token $FGA_TOKEN --config /dev/null
```

---

## After every operation

1. Show the raw output from the fga CLI
2. Show the exact command that was run (with all values filled in) in a code block labeled **Command run:**
3. If it was a `check`, display the result prominently as ✅ **ALLOWED** or ❌ **DENIED**
4. If it was a `write-model`, call out the new model ID prominently
5. If it was `write-tuples`, show the count of successfully written tuples
