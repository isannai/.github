<p align="center">
  <img src="./assets/hero.svg" alt="iSANN — Inference on nodes you're allowed on, execution inside a kernel sandbox" width="100%">
</p>

<p align="center">
  <b>Many nodes are linked across the iSANN network.</b><br>
  Of those, the only ones where inference and skills actually run are your own nodes<br>
  and the nodes opened to you, and that execution never leaves the kernel sandbox.
</p>

---

## Your own hardware is the agent runtime

You never hand your model to anyone else. Engines run in containers on your own GPU, and every tool an agent calls runs only inside a sandbox the kernel itself guards. The node you build this way is then called, as it is, by your other machines and by nodes you have access to.

| ⚙️&nbsp; **Engine lifecycle** | 🧠&nbsp; **Eight asset kinds, one contract** | 🛡️&nbsp; **Kernel-isolated execution** |
|---|---|---|
| Define llama.cpp, Stable Diffusion or vLLM with the four-file manifest convention and the daemon takes it from there: docker creation, startup, warm-up and status reporting. | From engines and models to skills and presets, all eight kinds share the same contract: a content-hash id and a provenance index. Where something came from and what changed stays traceable. | Tools only ever run inside a sandbox where the OS kernel itself blocks file and network access. The boundary sits one layer below the container. |
| `isann docker create llama` | `isann <kind> pull · list · inspect · rm` | `policy · deny-by-default` |

### Eight standard asset kinds

All under one contract: content-hash id, provenance index, `<kind>:<id>` reference syntax, and the same CLI.

| | | | |
|---|---|---|---|
| **engine**<br><sub>Inference engine definition<br>`manifest + compose + .env + run.sh`</sub> | **model**<br><sub>Weights<br>loaded from HuggingFace or local disk</sub> | **tool**<br><sub>A capability an agent calls<br>`tools.json`</sub> | **skill**<br><sub>Procedural knowledge for a situation<br>`SKILL.md`</sub> |
| **recipe**<br><sub>Declares the state a node should reach<br>`.ian`</sub> | **mesh**<br><sub>Backend apps a node runs<br>`mesh.json`</sub> | **preset**<br><sub>Inference parameter sets<br>keyed per engine</sub> | **profile**<br><sub>Engine runtime settings<br>`.env`</sub> |

**Plug them in, pull them out.** All eight attach with `pull` and detach with `rm`, and you never have to rebuild the node to do it.

Every one of them can be shared and traded between nodes. And two things go further still, carrying a price of their own: **one inference and one skill run**. With x402 you put a price on a single request, sell it to another node, and buy from theirs.

---

## Inference and skills both travel peer to peer

You type on the laptop and the desktop GPU answers. Nothing relays in between. The rendezvous only helps the two find each other; once the path opens, all that is left is a direct QUIC link between the nodes. The node on the other end may be another machine of yours, or one somebody opened to you. The route is identical either way. The only difference is what that node lets you do.

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="./assets/p2p-dark.svg">
    <img src="./assets/p2p-light.svg" alt="A laptop node finds its peer through a rendezvous server, then connects directly over QUIC to a desktop GPU node" width="100%">
  </picture>
</p>

| | |
|---|---|
| **1 · They find each other** | Nodes register with the rendezvous, and hole-punching opens a path even from behind NAT. That is the whole of what the rendezvous does. |
| **2 · They connect directly** | From then on the nodes speak QUIC to each other. Every request is signed with a wallet key to prove who is calling. |
| **3 · It runs right there** | Inference on that node's GPU, skills and tools in that node's kernel sandbox. Only the result streams back. |

```console
# register the desktop under an alias
$ isann favorite add desktop 0x9f3c…a71b
$ isann favorite use desktop

# inference: you ask from the laptop, the desktop GPU answers
$ isann infer --nodes desktop -m qwen3-8b -p "summarize this log"
  ✓ desktop  llama  1.9s  ↯ streaming

# skills: same path, executed inside the remote node's sandbox
$ isann agent run --nodes desktop --skill repo-audit
  ✓ sandbox  kernel-isolated  fs:ro  net:deny
```

### Skills your agent calls travel the very same path

The daemon ships with an MCP server built in. Attach any MCP-capable agent or editor to a node and the skills it calls **do not stop at your machine**. They cross to a remote node over P2P and run in that node's sandbox. From the agent's side it simply called a tool. All that changed is whose kernel it ran inside.

`MCP client` → `isannd (built-in MCP server)` → **`P2P · direct QUIC`** → `remote node sandbox`

---

## How this differs from things that look similar

Plenty of projects carry the words "distributed AI", and each of them is solving a different problem. What follows is not a claim that one is better than another. It is a map of where iSANN stands.

| | What it solves | Where inference runs | What it is good at | Where iSANN differs |
|---|---|---|---|---|
| **iSANN** | Making your own machines into one agent runtime | Your nodes and nodes you are allowed on | Engine management, kernel-isolated execution, direct node links | *The reference point* |
| **OpenClaw** | Driving a personal agent from a messenger | Mostly external model APIs (your own keys) | Deep channel integration and session handling | Runs the model on your node instead of calling an API |
| **Ollama / LM Studio** | Running models locally with little setup | One machine of yours | The simplest install and model management there is | Reaches past one machine, and covers tool execution too |
| **io.net / Vast.ai** | Renting as much GPU as you need | Rented GPUs | Large amounts of GPU available on demand | Uses hardware you own, and nothing leaves it |
| **Exo** | Running a large model across small devices | Split across devices | Fits a model no single machine could hold | Never splits a model; what moves between nodes is the request |
| **AI Horde** | Letting anyone run inference for free | An anonymous resource pool | No barrier to entry and no cost | A private mesh with identity and permissions attached |

**Your own machines, not rented ones.** This is not about renting a stranger's GPU. What you connect to is hardware you own, or a node somebody opened to you.

**Models are never split.** A single model is never spread across machines. One node runs one model whole, and what travels between nodes is the request.

**The kernel draws the line.** Tools that only do inference have no notion of execution at all, and agent gateways usually stop at the container. In iSANN the kernel itself draws the file and network boundary.

**Not an anonymous public pool.** Every request carries an identity signed with a wallet key, and the node's owner opens and closes the door by role (owner / admin / user).

---

## A node's identity is the machine itself

It is not an ID handed to you when you register an account. A node's address is computed from the mainboard and the security chip. There is no file to steal, and the address holds unless the machine physically changes.

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="./assets/nodeid-dark.svg">
    <img src="./assets/nodeid-light.svg" alt="Mainboard UUID and fTPM fingerprint are derived through HKDF-SHA256 into a secp256k1 key and a node EOA address" width="100%">
  </picture>
</p>

| | |
|---|---|
| **Swap the disk and it stays** | Reinstall the OS or replace the SSD and the address is unchanged, because identity here is computed rather than stored. |
| **There is no file to copy** | Steal a key file, move it to another machine, and it still is not that node. A different fingerprint yields a different address. |
| **Reputation attaches to the machine** | What a node has done belongs to one piece of hardware rather than a disposable account. Throwing it away and starting over costs real money. |
| **It still finds you when the IP changes** | What you call is the node id, not an IP. Move from home to a café, or let the router pick up a new address, and the same id still reaches it. You get what a static IP would give you without paying for one. |

> **The address only changes in two cases.** Replacing the mainboard changes the System UUID, and the address changes with it. Replacing the graphics card matters only for nodes derived from a GPU UUID because they have no security chip.
> Put the other way round: as long as you do not physically change the hardware, the address stays exactly where it is.

### iSANN is not a Web3 project

There is no chain to join and no token to hold, and nothing above this line waits on one. What we borrow from these standards is narrow and practical: **a way to prove which node is speaking, a way to hold payment until the work is done, and a way for a node's record to outlive its own word for it.** They are the wiring for the moment two strangers decide to trade.

| | | |
|---|---|---|
| **ERC-8004** · *Identity* <br>`Planned` | **ERC-8183** · *Escrow* <br>`Planned` | **x402** · *Per-request payment* <br>`Planned` |
| Registers the hardware-derived node address as an on-chain DID. Identity stops being something a node asserts about itself and becomes a record anyone can check. | Locks the payment up front and releases it once the work is done. The side that asked never loses money without a result, and the side that worked never goes unpaid. | Uses HTTP `402 Payment Required` exactly as it is. The caller never looks up a price list first. Call as you always would and the node answers with the price. |
| <sub>To hand work to a node you have never met, there has to be a shared ledger of what that node has done.</sub> | <sub>Between two nodes that do not know each other, a trade only happens if going first is not a risk.</sub> | <sub>Pricing by the request is what makes it possible to sell something as small as one skill run.</sub> |

**No token is used to fence the ecosystem in.** Identity, escrow and payment all sit on standards that already exist, so anything else speaking them connects straight through.

A **credit** may still be issued to settle node usage. That credit is made to be spent, not held, so a substantial share of it is **burned** every time it moves to someone else. The more hands it passes through, the less of it remains, which leaves the value in actual use rather than in a price.

---

## Repositories

One runtime distribution, one backend, and the asset repositories that nodes pull from. Of the eight asset kinds only **model** is absent here, because weights come straight from HuggingFace or local disk.

| | |
|---|---|
| **[isann](https://github.com/isannai/isann)** · `runtime`<br>The runtime distribution. Daemon, CLI and version manager (ivm) binaries ship here as releases. Start from this one.<br><sub>`ivm install 0.1.2`</sub> | **[engines](https://github.com/isannai/engines)** · `engine`<br>Inference engine bundles. llama.cpp, Stable Diffusion and vLLM, each defined by the four-file convention.<br><sub>`isann engine pull github.com/isannai/engines/tree/main/llama`</sub> |
| **[skills](https://github.com/isannai/skills)** · `asset`<br>Procedural knowledge an agent loads when the situation calls for it (`SKILL.md`), together with the tool references it needs.<br><sub>`isann skill pull github.com/isannai/skills/tree/main/<name>`</sub> | **[tools](https://github.com/isannai/tools)** · `asset`<br>Tool definitions (`tools.json`). The unit an agent actually calls, and running one always has to clear the sandbox policy first.<br><sub>`isann tool pull github.com/isannai/tools/tree/main/<name>`</sub> |
| **[recipes](https://github.com/isannai/recipes)** · `asset`<br>Brings a single node to the state you want, from starting engines to preparing models and placing skills.<br><sub>`isann recipe pull github.com/isannai/recipes/tree/main/<name>`</sub> | **[presets](https://github.com/isannai/presets)** · `config`<br>Bundle temperature, top-p, context and the like under one name per engine, and requests pick them up on their own.<br><sub>`isann preset pull github.com/isannai/presets/tree/main/<name>`</sub> |
| **[profiles](https://github.com/isannai/profiles)** · `config`<br>Engine runtime profiles (`.env`). For running the same engine several ways by changing only GPU memory, batch size or port.<br><sub>`isann profile pull github.com/isannai/profiles/tree/main/<name>`</sub> | **[mesh](https://github.com/isannai/mesh)** · `backend`<br>The station and control apps that let nodes find each other and surface as services live here, and the daemon starts them.<br><sub>`isann mesh start station`</sub> |

---

## Four steps to your first inference

| | | | |
|---|---|---|---|
| **01** Prepare the node | **02** Install the runtime | **03** Start an engine | **04** Infer |
| `ivm init && ivm setup` | `ivm install 0.1.2 && ivm use 0.1.2` | `isann docker create llama` | `isann infer -m qwen3-8b -p "hello"` |

---

## Roadmap

No dates here, deliberately. The order is settled; the timing is not.

<table>
<tr>
<td width="50%" valign="top">

**`Now`  Working on day one**

<sub>Infrastructure and the developer tooling around it.</sub>

- **Engine lifecycle** over docker: llama.cpp, Stable Diffusion, vLLM
- **Direct node-to-node links**: QUIC, NAT hole-punching, cross-node commands
- **Node identity derived from hardware**, with every request signed
- A **built-in MCP server**, plus agent, tool and skill execution
- **Eight asset kinds** behind a single contract
- The **ivm** version manager: install, switch, service, use

</td>
<td width="50%" valign="top">

**`Next`  Designed, not yet written**

<sub>What it takes to share a node with somebody else.</sub>

- **Multiple rendezvous**: more than one place for nodes to find each other, so one going down does not stop them meeting.
- **ERC-8004 · ERC-8183 · x402**: widen the place nodes are found into a market (called by an ENS name rather than `0x9f3c…`), put a price on a single request, and hold the payment until the work is done. You cannot sell what nobody can find and you cannot buy what you cannot trust, so **the three open together or not at all.**
- **Credit faucet**: a small handful of credits for newcomers, so you can try the thing before opening a wallet.
- **Asset snapshots**: capture everything installed on a node as one thing, then bring it back or hand it to another node.

</td>
</tr>
</table>

---

<p align="center">
  <sub><b>iSANN</b> · Interstellar Artificial Neural Network &nbsp;·&nbsp; MIT License &nbsp;·&nbsp; Go 1.25 &nbsp;·&nbsp; Windows · Linux</sub>
</p>
