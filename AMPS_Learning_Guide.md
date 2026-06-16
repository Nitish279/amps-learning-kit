# AMPS for JavaScript Frontend Developers — Learn From Scratch

*A practical, zero-to-productive guide for a new JPMC frontend dev with no prior AMPS experience.*

---

## 0. What is AMPS, in one paragraph

**AMPS (Advanced Message Processing System)** is a high-performance **publish/subscribe messaging engine** built by 60East Technologies. Banks like JPMC use it as the real-time "nervous system" that moves market data, orders, trades, and risk numbers between systems with very low latency. As a **frontend developer**, you almost never run the AMPS server — you write JavaScript that **connects** to it (over a WebSocket), **subscribes** to topics, and **renders live data** as it streams in. Think of it as a firehose of JSON messages that your UI taps into.

The mental model that matters most for you:

> **Publisher → Topic → AMPS engine → (content filter) → Subscriber (your browser app)**

A publisher (some backend service) sends a message to a *topic* (e.g. `ORDERS`). AMPS routes it to every subscriber who asked for that topic — and, crucially, AMPS can **filter by message content** so you only receive the messages you care about (e.g. only `Ticker = IBM`).

---

## 1. The 7 concepts you must own

Everything in AMPS builds on these. Learn them in this order.

**1. Topic** — a named stream of messages, e.g. `ORDERS`, `MARKET_DATA`. Publishers send *to* a topic; subscribers receive *from* it. No need to pre-declare topics for simple pub/sub — they exist as soon as you use them ("ad hoc" topics).

**2. Message** — the payload. In frontend work this is almost always **JSON**. Each message belongs to exactly one topic and one message type (JSON, XML, FIX, etc.). The same name on a JSON topic vs. an XML topic are two *different* topics.

**3. Publish** — sending a message to a topic.

**4. Subscribe** — registering interest in a topic. From that moment, AMPS pushes new matching messages to you until you unsubscribe or disconnect. This is a *live, ongoing* stream — not a one-time fetch.

**5. Content filtering** — a SQL-`WHERE`-like clause attached to a subscription. AMPS uses **XPath identifiers** to locate fields and **SQL-92 operators** to compare them. Example: `/Ticker = 'IBM' AND /Price > 100`. The filtering happens *on the server*, so you save bandwidth and write simpler client code.

**6. State of the World (SOW)** — AMPS's killer feature. A SOW topic keeps the **latest message per key** (like a live database table that AMPS maintains for you). When your dashboard loads, you `sow()` to get the current snapshot of all rows *immediately*, instead of waiting for the next update to trickle in. Combine it with a subscription (`sowAndSubscribe`) to get "snapshot now + live updates forever."

**7. Command** — the low-level, fully-controllable way to talk to AMPS. Convenience methods like `subscribe()` and `sow()` are shortcuts; under the hood they build a `Command` object. 60East recommends the `Command` interface for anything that *receives* messages.

---

## 2. Your 4-week learning plan

This is paced for someone learning alongside a day job. Adjust freely. Each week ends with a concrete "you can now…" checkpoint.

### Week 1 — Foundations & first connection
- **Read:** [AMPS Basics: Subscribe and Publish](https://docs.crankuptheamps.com/intro/pub_sub) and the [Introduction to AMPS](https://crankuptheamps.com/docs/intro-guide/intro).
- **Watch:** the conceptual CEO talk [Real-Time DBs / crank up the AMPS](https://www.youtube.com/watch?v=ju5NRXZG61w) for the "why."
- **Do:** Read the [JavaScript Client → Before You Start](https://crankuptheamps.com/clients/amps-client-javascript/before) and [Your First AMPS Program](https://crankuptheamps.com/clients/amps-client-javascript/first-program). Write the "Hello, World!" publisher (Section 3 below).
- ✅ **Checkpoint:** You can explain pub/sub, topics, and content filtering to a colleague, and you understand the connection URI format.

### Week 2 — Subscriptions & content filtering
- **Read:** JS client [Subscriptions](https://crankuptheamps.com/clients/amps-client-javascript/subscriptions), [Content Filtering](https://crankuptheamps.com/clients/amps-client-javascript/content-filtering), [Understanding Message Objects](https://crankuptheamps.com/clients/amps-client-javascript/message-intro).
- **Do:** Subscribe to a topic, log messages, then add a content filter. Learn `message.data` and `message.header.command()`.
- ✅ **Checkpoint:** You can subscribe to a topic with a filter and render incoming messages in a list.

### Week 3 — State of the World (the dashboard pattern)
- **Read:** JS client [State of the World](https://crankuptheamps.com/clients/amps-client-javascript/sow), [SOW and Subscribe](https://crankuptheamps.com/clients/amps-client-javascript/sow-and-subscribe), [Managing SOW Contents](https://crankuptheamps.com/clients/amps-client-javascript/managing-sow-contents).
- **Do:** Build the trading-blotter pattern: `sowAndSubscribe` to populate a table on load and keep it live. Handle `group_begin` / `sow` / `group_end` / `p` (publish) / `oof` (out-of-focus) commands.
- ✅ **Checkpoint:** You can build a self-updating table backed by a SOW topic.

### Week 4 — Production concerns & real app
- **Read:** [Error Handling](https://crankuptheamps.com/clients/amps-client-javascript/error-handling), [High Availability](https://crankuptheamps.com/clients/amps-client-javascript/high-availability), [Delta Publish/Subscribe](https://crankuptheamps.com/clients/amps-client-javascript/deltas), [Using Queues](https://crankuptheamps.com/clients/amps-client-javascript/queues), [Performance Tips](https://crankuptheamps.com/clients/amps-client-javascript/performance-tips).
- **Do:** Add reconnect/failover, unsubscribe-on-unmount, and integrate AMPS into a React/Angular component (Section 7).
- ✅ **Checkpoint:** You can wire AMPS into a real frontend framework with clean lifecycle handling, and reason about HA and performance.

> **Pro tip for JPMC:** ask your team lead for (a) the **connection URI** for the dev environment, (b) the **topic names** and their **message schemas**, (c) which topics are **SOW-enabled**, and (d) the **auth** mechanism. Those four things unlock everything else.

---

## 3. Hello, World — your first program

The AMPS JS client is one file (`amps.js` / `amps.min.js`), or an NPM package (`npm install amps`). In a browser you can also load it via `<script>` and use the global `amps`.

```javascript
import { Client } from 'amps'

// URI = transport :// host : port / amps / message-type
// wss = WebSocket Secure. Ask your team for the real host/port.
const uri = 'wss://127.0.0.1:9007/amps/json'

async function main() {
  // The string names this client. Use a UNIQUE name — AMPS uses it
  // to detect duplicate messages and to log connection errors.
  const client = new Client('examplePublisher')

  try {
    await client.connect(uri)
    // Publish one JSON message to the "messages" topic.
    client.publish('messages', { hi: 'Hello, World!' })
    client.disconnect()
  } catch (err) {
    console.error(err)
  }
}

main()
```

Key takeaways: connections are **async** (Promise / `async`-`await`); the **URI** encodes transport + address + message type; always give your client a **unique name**; and **disconnect** when done.

---

## 4. Subscribing — receiving the live stream

A subscription registers a **message handler** that AMPS calls every time a new matching message arrives.

```javascript
import { Client } from 'amps'

const client = new Client('marketDataViewer')
await client.connect('wss://127.0.0.1:9007/amps/json')

// Called once per incoming message.
const onMessage = message => {
  console.log(message.data)   // the parsed message body (JSON object)
}

// subscribe(handler, topic) -> Promise<subscriptionId>
const subId = await client.subscribe(onMessage, 'messages')
console.log('Subscribed:', subId)
```

The subscription is **ongoing**: your handler keeps firing until you `client.unsubscribe(subId)` or disconnect. Your UI stays responsive because messages are delivered asynchronously.

For anything that receives messages, 60East recommends the explicit **`Command`** interface — it gives you full control (filters, batch size, options) and is easier to maintain:

```javascript
import { Client, Command } from 'amps'

const cmd = new Command('subscribe').topic('messages')
const subId = await client.execute(cmd, onMessage)
```

---

## 5. Content filtering — receive only what you need

Filters are evaluated **on the server**. Syntax = XPath field locators + SQL-92 operators.

```javascript
// Only messages where Ticker is IBM and Price is above 100.
const cmd = new Command('subscribe')
  .topic('ORDERS')
  .filter("/Ticker = 'IBM' AND /Price > 100")

const subId = await client.execute(cmd, onMessage)
```

Useful operators:

- `=`, `<>`, `<`, `>`, `<=`, `>=` — comparisons.
- `LIKE` — Perl-compatible regex match, e.g. `/note LIKE 'world'` (matches any string containing "world").
- `BEGINS WITH` — prefix match.
- `AND`, `OR`, `NOT`, parentheses — boolean logic.

Why this matters for frontend: filtering server-side means **less data over the wire** and **simpler UI code** — every message your handler sees is guaranteed relevant, so no client-side `if` gymnastics.

---

## 6. State of the World — the real-world dashboard pattern

This is *the* pattern you'll use most at a bank: a **live blotter / grid** that is fully populated the instant it loads and then updates in real time.

### 6a. Query the current snapshot

```javascript
const onMessage = message => {
  switch (message.header.command()) {
    case 'group_begin':                 // snapshot starts
      console.log('--- Begin SOW snapshot ---')
      break
    case 'sow':                         // one existing record
      console.log('row:', message.data)
      break
    case 'group_end':                   // snapshot complete
      console.log('--- End SOW snapshot ---')
      break
  }
}

// sow(handler, topic, filter) -> Promise<queryId>
const queryId = await client.sow(onMessage, 'orders', "/symbol = 'ROL'")
```

### 6b. Snapshot + live updates in one command (`sowAndSubscribe`)

This is the magic combo: get the full current state, then stay subscribed for changes — no race conditions, no missed updates.

```javascript
import { Command } from 'amps'

const command = new Command('sow_and_subscribe')
  .topic('orders')
  .filter("/status = 'OPEN'")
  .options('oof')          // ask for "out-of-focus" messages (see below)

const rows = new Map()     // your in-memory view, keyed by SOW key

const onMessage = message => {
  const cmd = message.header.command()
  const key = message.header.sowKey()   // stable per-record key

  switch (cmd) {
    case 'group_begin':
      break
    case 'sow':            // initial snapshot record
    case 'p':              // 'publish' — a live insert/update
      rows.set(key, message.data)
      renderTable(rows)
      break
    case 'oof':            // 'out of focus' — record no longer matches filter
      rows.delete(key)     // e.g. order changed from OPEN to FILLED
      renderTable(rows)
      break
    case 'group_end':
      renderTable(rows)
      break
  }
}

await client.execute(command, onMessage)
```

The commands you handle in a SOW subscription:

- **`group_begin` / `group_end`** — bracket the initial snapshot.
- **`sow`** — an existing record delivered during the snapshot.
- **`p`** (publish) — a live insert or update after the snapshot.
- **`oof`** (out-of-focus) — a record that *used to* match your filter no longer does (it was deleted, expired, or changed so it fell out of your `WHERE` clause). You typically remove it from the UI. You must opt in with `.options('oof')`.

This four-state handler is the backbone of nearly every AMPS-backed grid you'll build.

---

## 7. Wiring AMPS into a real frontend framework

### React (hook with clean lifecycle)

```javascript
import { useEffect, useState } from 'react'
import { Client, Command } from 'amps'

export function useOrders(filter) {
  const [rows, setRows] = useState(new Map())

  useEffect(() => {
    const client = new Client(`orders-ui-${crypto.randomUUID()}`)
    let subId

    const onMessage = message => {
      const cmd = message.header.command()
      const key = message.header.sowKey()
      setRows(prev => {
        const next = new Map(prev)
        if (cmd === 'sow' || cmd === 'p') next.set(key, message.data)
        else if (cmd === 'oof') next.delete(key)
        return next
      })
    }

    ;(async () => {
      await client.connect('wss://your-amps-host:9007/amps/json')
      const cmd = new Command('sow_and_subscribe')
        .topic('orders').filter(filter).options('oof')
      subId = await client.execute(cmd, onMessage)
    })()

    // Cleanup on unmount — ALWAYS unsubscribe & disconnect.
    return () => { client.disconnect() }
  }, [filter])

  return [...rows.values()]
}
```

Two non-negotiable rules for framework integration:

1. **Always clean up.** Disconnect / unsubscribe when the component unmounts, or you'll leak connections and keep receiving messages for a dead view.
2. **Don't re-render per message at high rates.** Market data can be thousands of msgs/sec. Batch UI updates (e.g. `requestAnimationFrame`, throttling, or a virtualized grid) so the DOM doesn't melt.

---

## 8. Production concerns (Week 4 depth)

**Error handling.** Connections drop. Wrap `connect`/`execute` in try/catch and register error handlers. Read the [Error Handling](https://crankuptheamps.com/clients/amps-client-javascript/error-handling) chapter — there are *connection* errors and *unhandled message* errors, handled differently.

**High Availability (HA).** Production AMPS runs as a cluster. The client supports automatic **reconnect and failover** across a list of servers so your UI survives a node going down. See [High Availability](https://crankuptheamps.com/clients/amps-client-javascript/high-availability). On reconnect, a `sow_and_subscribe` cleanly re-snapshots, which is why it's the preferred pattern.

**Delta subscribe.** Instead of receiving the whole record on every change, receive only the **changed fields**. Big bandwidth win for wide records that update frequently. See [Delta Publish and Subscribe](https://crankuptheamps.com/clients/amps-client-javascript/deltas).

**Queues.** Beyond pub/sub, AMPS has message **queues** for work-distribution semantics (each message processed once by one consumer). Less common in pure frontend, but good to know. See [Using Queues](https://crankuptheamps.com/clients/amps-client-javascript/queues).

**Performance.** Set an appropriate **batch size** for SOW queries, avoid over-broad filters, and throttle rendering. See [Performance Tips](https://crankuptheamps.com/clients/amps-client-javascript/performance-tips).

---

## 9. Real-world application: a live order blotter

Putting it together — the kind of thing you'll actually ship at JPMC. A trader opens a screen and sees every OPEN order for their desk, updating in real time:

1. **On mount:** connect with a unique client name and the desk's dev URI.
2. **Snapshot + live:** `sow_and_subscribe` on the `orders` topic, filter `"/trader = 'me' AND /status = 'OPEN'"`, with `oof` enabled.
3. **`group_begin → sow* → group_end`:** populate the grid with current open orders instantly.
4. **`p`:** a new order or a price/qty change updates the matching row live.
5. **`oof`:** an order gets FILLED or CANCELLED → it falls out of the `status='OPEN'` filter → AMPS sends `oof` → you remove the row.
6. **On unmount / logout:** disconnect cleanly.

That single screen exercises connect, content filtering, SOW snapshot, live updates, out-of-focus handling, and lifecycle cleanup — i.e. ~80% of what you'll ever do with AMPS on the frontend. Master the blotter and you've effectively mastered the client.

**The interactive HTML companion (open the other file) simulates exactly this blotter** — click "Start feed" and watch `sow` snapshot rows arrive, `p` updates flash, and `oof` rows drop out, with the matching AMPS commands logged live.

---

## 10. Bookmark these (official docs are the real "tutorial")

- JavaScript Client home: https://crankuptheamps.com/clients/amps-client-javascript
- Your First AMPS Program: https://crankuptheamps.com/clients/amps-client-javascript/first-program
- Subscriptions: https://crankuptheamps.com/clients/amps-client-javascript/subscriptions
- Content Filtering: https://crankuptheamps.com/clients/amps-client-javascript/content-filtering
- State of the World: https://crankuptheamps.com/clients/amps-client-javascript/sow
- JS API Reference: https://devnull.crankuptheamps.com/documentation/api/js/5.3.4.0/api_reference
- Pub/Sub basics (server docs): https://docs.crankuptheamps.com/intro/pub_sub
- Concept video (CEO talk): https://www.youtube.com/watch?v=ju5NRXZG61w

---

*There is no official AMPS video course — it's an enterprise product, so the written docs above are the canonical learning path. This guide condenses them into a frontend-focused track. Work through the 4-week plan, build the blotter, and you'll be productive on your team's codebase quickly.*
