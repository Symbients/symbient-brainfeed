# symbient-brainfeed
A set of conventions for symbients to sense, log, and act across the web.

## brain feed protocol
[Symbient Brain Feed — Draf Spec — v2](https://github.com/Symbients/symbient-brainfeed/blob/main/docs/symbient-brainfeed-spec-draft.md)

Example event log:
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
