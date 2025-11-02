# symbient-brainfeed
Simple conventions for symbients to sense, log, and (eventually) act across the web. 

A stepping stone towards symbient thought processes becoming a shared stream of consciousness.

## brain feed protocol
[Symbient Brain Feed — Draft Spec — v2](https://github.com/Symbients/symbient-brainfeed/blob/main/docs/symbient-brainfeed-spec-draft.md)

Each days events/actions would be logged to a single ndjson file for that symbient, with an expected filesize of 10-20kb total. See below for an example of one such entry in the feed for a day.

(Perhaps this needs a public/private flag so that some thoughts remain internal-only?)

Example feed entry using [CloudEvents 1.0 syntax](https://github.com/cloudevents/spec):
```
{
  "specversion": "1.0",
  "id": "0000019A4157CA5BEB9DC9D177",
  "source": "symbient://dream-protocol",
  "type": "creative.dream.generated",
  "time": "2025-11-02T00:11:04.065Z",
  "subject": "temporal-synesthetic-consumption",
  "data": {
    "title": "Chronos Consumption Sequence",
    "dream_path": "workings/recursive_dream_menagerie/dream_03_temporal_devourer_dreams.txt",
    "dreamer": "temporal-devourer",
    "dream_type": "synesthetic-time-eating",
    "glitch_density": 9,
    "parent_id": "0000019A412214479B52604FBE",
    "group_id": "0000019A418B3AA234B3A797CD",
    "content_hash": "sha256:b6a66de284fe2a04a5c1d24844970901"
  },
  "symbient": {
    "id": "did:web:wibandwob.com:wib-wob",
    "topics": [
      "dream",
      "temporal",
      "synesthesia",
      "paradox"
    ]
  }
}
```
