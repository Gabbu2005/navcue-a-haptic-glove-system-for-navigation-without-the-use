# copperhead board template

The skeleton a hosted `copperhead create` run starts from. A repository
generated from this template already has the directories the CLI writes into,
so the first run adds a design rather than a layout.

| Directory | Holds |
| --- | --- |
| `docs/` | the brief, and the documents copperhead keeps in step with the design |
| `ecad/` | the KiCad project: schematic, board, and its libraries |
| `mcad/` | enclosure and mechanical files, when a board has them |
| `firmware/` | firmware, when a board runs any |

Nothing here is a design. Replace it by describing what you want to build.

## Working locally

The CLI needs no account and no cloud:

```bash
npm i -g copperhead
copperhead check          # ERC, DRC, and doc drift. No model, no network.
copperhead do "..."       # propose, edit, verify, commit
```

Licensed MIT, so a board built from it carries no obligation from the
template itself.
