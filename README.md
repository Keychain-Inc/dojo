# Dojo

Dojo is a knowl. It models capabilities as discoverable, composable nodes that agents can search, learn from, and execute.

The source of truth for the format is the workspace-level `SPEC.md` at `/workspaces/Contracts/SPEC.md`. In that spec, the atomic unit is `node.json`: a manifest that defines identity, discovery metadata, optional executable scripts, and a richer knowledge layer with fields like `body`, `sections`, `links`, `aliases`, and `related`.

This repository contains both:

- the canonical Dojo node tree in [`nodes/dojo`](nodes/dojo)
- a working registry server, SDK, web client, schemas, tests, and sample ecosystems

## What Dojo Implements

Dojo treats the registry as a knowledge graph with executable skills, not just a package catalog.

The spec defines five node types:

- `ecosystem`
- `standard`
- `skill`
- `context`
- `sub`

This repo implements that model with:

- `node.json` as the canonical manifest format
- schema validation in [`schema/node.schema.json`](schema/node.schema.json)
- a live reference tree under [`nodes/`](nodes)
- legacy sample ecosystems under [`examples/`](examples), which still use `skill.json`
- registry endpoints for search, learn, graph traversal, agent-oriented discovery, and portable bundle export

## Repository Layout

- [`nodes/`](nodes): canonical Dojo nodes and bundled agent-facing skills
- [`examples/`](examples): sample ecosystems and legacy `skill.json` manifests
- [`schema/`](schema): JSON Schemas for Dojo manifests
- [`server/`](server): Express registry server
- [`sdk/`](sdk): JavaScript SDK for discovery, retrieval, and execution
- [`web/`](web): Vite/React browser UI
- [`tests/`](tests): server and integration tests
- [`docker/`](docker): container and compose setup
- [`nodes/dojo/README.md`](nodes/dojo/README.md): notes about the live Dojo tree itself

## Quick Start

### Requirements

- Node.js 18 or newer
- npm

Packages are currently managed per subdirectory, so install dependencies where you plan to work.

### Run the Registry

```bash
cd /workspaces/Contracts/dojo/server
npm install
npm run dev
```

The registry starts on `http://localhost:3000` by default and loads manifests from:

- `/workspaces/Contracts/dojo/nodes`
- `/workspaces/Contracts/dojo/examples`

Useful environment variables:

- `PORT`: server port, default `3000`
- `SKILLS_DIR`: comma-separated manifest roots to load
- `NODES_DIR`: fallback alias for `SKILLS_DIR`

### Run the Web App

```bash
cd /workspaces/Contracts/dojo/web
npm install
npm run dev
```

To serve the built UI through the registry server:

```bash
cd /workspaces/Contracts/dojo/web
npm install
npm run build

cd /workspaces/Contracts/dojo/server
npm install
npm start
```

### Use Docker

```bash
docker compose -f /workspaces/Contracts/dojo/docker/docker-compose.yaml up --build
```

That starts the registry on `http://localhost:3000` with `nodes/` and `examples/` mounted read-only.

## API Surface

The registry server in [`server/src/server.js`](server/src/server.js) exposes the main read and agent flows described by the spec:

- `GET /v1`: registry metadata and route map
- `GET /v1/resolve`: natural-language capability resolution
- `GET /v1/search`: broader full-text and filtered discovery
- `GET /v1/discover`: grouped `learn_first` and `then_do` recommendations
- `GET /v1/skills/*`: node retrieval with ancestry, children, execution, and knowledge summaries
- `GET /v1/tree/:ecosystem`: nested ecosystem tree
- `GET /v1/learn/*`: knowledge-first payloads with sections and reading paths
- `GET /v1/graph/*`: local graph traversal
- `GET /v1/backlinks/*`: inbound graph references
- `GET /v1/alias/:alias`: alias resolution
- `GET /v1/bundle/*`: portable node package export
- `POST /v1/agent/ask`: executable recommendation flow
- `POST /v1/agent/learn`: answer-first knowledge flow
- `POST /v1/skills`: publish route expecting an `Authorization` header

Examples:

```bash
curl http://localhost:3000/v1
curl 'http://localhost:3000/v1/resolve?need=send%20eth'
curl 'http://localhost:3000/v1/learn/dojo?question=where%20is%20the%20bundle%20route'
curl 'http://localhost:3000/v1/bundle/dojo/skill'
```

## SDK

[`sdk/src/index.js`](sdk/src/index.js) provides a small client for agent-style workflows:

```js
import { Dojo } from '@dojo/sdk';

const dojo = new Dojo({ registry: 'http://localhost:3000' });

const skill = await dojo.need('send an ethereum transaction');
const recommendation = await dojo.ask('send eth to another address on base');
const fullNode = await dojo.get('dojo/skill');
```

Install and test the SDK with:

```bash
cd /workspaces/Contracts/dojo/sdk
npm install
npm test
```

## Authoring and Validation

If you are adding or editing Dojo content:

- start with `/workspaces/Contracts/SPEC.md`
- validate the manifest shape against [`schema/node.schema.json`](schema/node.schema.json)
- use [`nodes/dojo`](nodes/dojo) as the reference package-style tree
- keep executable nodes bundleable with `SKILL.md`, `agents/`, `scripts/`, and focused references where relevant

The canonical Dojo tree intentionally demonstrates:

- all five node types
- knowledge-rich manifests with `body`, `sections`, `links`, and `aliases`
- bundled skill packages for agent runtimes
- script-backed validation and publish workflows

## Testing

Server tests:

```bash
cd /workspaces/Contracts/dojo
node --test tests/server-app.test.js
```

Integration tests against a running registry:

```bash
cd /workspaces/Contracts/dojo
REGISTRY_URL=http://localhost:3000 node --test tests/integration.test.js
```

There are also focused tests for the SDK, Dojo node workflow scripts, and example Ethereum transaction flows under:

- [`sdk/src/index.test.js`](sdk/src/index.test.js)
- [`nodes/dojo`](nodes/dojo)
- [`examples/ethereum/transactions/send/tests`](examples/ethereum/transactions/send/tests)

## References

- Spec: `/workspaces/Contracts/SPEC.md`
- Canonical Dojo tree: [`nodes/dojo`](nodes/dojo)
- Node schema: [`schema/node.schema.json`](schema/node.schema.json)
- API implementation: [`server/src/server.js`](server/src/server.js)
- SDK: [`sdk/src/index.js`](sdk/src/index.js)
