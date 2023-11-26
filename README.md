> early alpha, written without trying to run it once. Don't use this yet. 

# nostr-trawl 
> Trawl [/trɔːl/] 
> 1. an act of fishing with a trawl net or seine.
> 2. a thorough search.

`nostr-trawl` is a simple tool for persistently fetching and processing filtered events from a set of [Nostr](https://nostr.io) relays.

## Queue Adapters
- `BullMQAdapter`: Persistent queue
- `PQueueAdapter`: Ephemeral javascript queue

## Usage 
_Theoretical usage, tests and implementation are incomplete_ 

```js
import { createTrawler } from '../src/index.js'

const relays = [ 'wss://relay.damus.io', 'wss://relay.snort.social' ]

const event_ids = new Set()

const options = {
  filters: { kinds: [3] },
  adapter: 'bullmq',
  queueName: 'ContactLists',
  repeatWhenComplete: true,
  restDuration: 60*60*1000,
  relaysPerBatch: 3,
  nostrFetchOptions: {
    sort: true
  },
  adapterOptions: {
    redis: {
      host: 'localhost',
      port: 6379, 
      db: 0
    }
  },
  queueOptions: {
    removeOnComplete: true, 
    removeOnFail: true
  },
  parser: async (event) => {
    event_ids.add(event.id)
    // console.log(event.created_at)
  },
  validator: (event) => {
    if(event_ids.has(event.id))
      return false 
    return true
  } 
}

const trawler = createTrawler(relays, options)

trawler
  .on_worker('completed', (job) => console.log(`${job.data.relay}: completed jobn`, 'data:', job))
  .on_worker('progress', (job, progress) => console.log(`[chunk #${progress.last_timestamp}] ${progress.relay}: ${progress.found} events found and ${progress.rejected} events rejected`))
  .on_queue('drained', () => console.log(`queue is empty`))

trawler.run()
```