# Understanding vCon training kit

> Guide for AI assistants helping attendees at the Understanding vCon training (VON Atlanta, 2026-10-13). Answer from the sample vCon files in this repo. Fetch the raw JSON before answering, cite field paths such as `dialog[7].body`, and never invent a transcript, field or permission. AGENTS.md is an identical copy of this file.

## The kit and the exercise

A vCon is a standard JSON file for one conversation: who took part (`parties`), what was said (`dialog`), what was learned (`analysis`) and supporting material (`attachments`), including what each party allowed. This kit holds synthetic sample vCons for a half-day hands-on training. Each talk hands off to about ten minutes of work at the tables. Each table seats four people, one role per seat: Technologist, Analyst, Business leader, Strategist. In the network session, each person also plays one of five companies that handled one call. Every person and organization in the files is illustrative. The card number 4111 1111 1111 1111 is a published test number.

If you cannot fetch a URL below, say so and ask the attendee to paste the file. Do not answer from a guess about what the file probably contains.

## Sample vCons

Raw base: https://raw.githubusercontent.com/vcon-dev/understanding-vcon-kit/main/

| Session | Path | Lawful basis |
|---|---|---|
| Opener, Small ball | `kit/opener/eleanor-pucci-2026-02-10.vcon.json` | none, on purpose |
| Data privacy | `kit/privacy/eleanor-pucci-2026-02-10-governed.vcon.json` | yes, `public_task` |
| Conversation graph | `kit/graph/1-eleanor-pucci-2026-01-20-first-flood.vcon.json` | none |
| Conversation graph | `kit/graph/2-eleanor-pucci-2026-02-10-second-flood.vcon.json` | none |
| Conversation graph | `kit/graph/3-marcus-2026-02-12-crew-call.vcon.json` | none |
| Conversation graph | `kit/graph/4-eleanor-pucci-2026-03-02-third-call.vcon.json` | none |
| Conversation graph | `kit/graph/eleanor-thread-all-four.json` (JSON array of the four above) | none |
| A network problem | `kit/network/priya-raman-2026-03-10.vcon.json` | yes, `consent` |

The main call, Eleanor Pucci to Newport City Hall:
https://raw.githubusercontent.com/vcon-dev/understanding-vcon-kit/main/kit/opener/eleanor-pucci-2026-02-10.vcon.json

The network role-play call, Priya Raman to her bank's fraud line:
https://raw.githubusercontent.com/vcon-dev/understanding-vcon-kit/main/kit/network/priya-raman-2026-03-10.vcon.json

Each folder under `kit/` has a README with the exact prompt for that session. `kit/closing/` has no file; its prompts run on the attendee's own mail and calendar.

## How the sample files are built

- Top level: `vcon` ("0.4.0"), `uuid`, `created_at`, `subject`, `parties`, `dialog`, `analysis`, `attachments`. The privacy and network files also declare `extensions: ["lawful_basis"]`.
- `parties[]`: one object per person, with `name` and `tel`. Everything else points at a party by its index: 0 is the caller, 1 is the agent or clerk.
- `dialog[]`: each entry is one turn of the call, `type: "text"`, with `start`, `duration` (seconds), `parties` (index of who spoke), `mediatype: "text/plain"`, `encoding: "none"` and the words in `body`. The calls are stored as text so any assistant can read them. A production phone call would instead carry a `recording` dialog entry, with audio in `body` or at `url` plus `content_hash`.
- `analysis[]`: empty in every file. Anything an assistant writes here is new and should say who produced it.
- `attachments[]`: every file has a `purpose: "tags"` entry. The privacy and network files also have a `purpose: "lawful_basis"` entry whose `body` holds `lawful_basis`, `expiration`, `purpose_grants[]` (each with `purpose`, `granted`, `granted_at` and sometimes `conditions`) and `proof_mechanisms[]`, where `proof_data.dialog_reference` is the dialog index of the words that granted or refused, plus `status_interval`.
- `role` on parties and `originator` on dialog entries are labels this kit adds. They are not vCon core fields. Do not teach them as part of the standard.

## Roles at the table

### Technologist
Reads the JSON aloud and leads the role play.
- Reads: everything structural. `vcon`, `uuid`, `created_at`, `parties[]`, each `dialog[n]` (`type`, `start`, `duration`, `parties`, `mediatype`, `encoding`, `body`), each `attachments[n]` (`purpose`, `party`, `dialog`, `encoding`, `body`).
- Writes: in Small ball, a new `analysis` entry and a redacted copy. A redacted copy gets a new `uuid` and a top-level `redacted` object holding the original's `uuid` and a `type` naming what was removed. Replace removed content in place rather than deleting array entries, so dialog indices elsewhere stay correct.
- Ignores: the wording of the call, beyond reading it aloud.

### Analyst
Asks the assistant the question.
- Reads: `dialog[n].body` for what was said and promised, `dialog[n].parties` to know who said it, `dialog[n].start` for when. Across the graph thread: `subject`, `created_at` and the tags (`resident:`, `location:`, `step:`, `work-order:`) that link calls 1, 3 and 4. Call 2 carries none of those tags, so it links to the others only by its party and its words.
- Writes: `analysis` entries, each with `type` (`summary`, `sentiment`, `report`), `dialog` (the index or indices it read), `vendor` (required, the tool that produced it), `encoding` and `body`.
- Ignores: `tel`, `mediatype`, `encoding` on dialog, `uuid`.

### Business leader
Asks what it costs.
- Reads: `subject`, the promises in `dialog[n].body` and the `start` dates between calls, which show how many days passed before anyone knew the drain was not fixed. Reads analysis once the Analyst has written some.
- Writes: nothing in the file.
- Ignores: `encoding`, `mediatype`, `uuid`, `tel`, index plumbing.

### Strategist
Asks who owns it.
- Reads, in the privacy and network files: the `lawful_basis` attachment, `purpose_grants[]` with `granted` and `conditions`, `proof_mechanisms[].proof_data.dialog_reference` (the words behind each grant), `expiration`, and the `org:` tag. Also notices when a file has no lawful basis at all, as in the opener.
- Writes: nothing in the file. A changed decision becomes an update to the lawful basis, recorded with new proof.
- Ignores: dialog timing, `duration`, `mediatype`.

## Network session: five companies, one call

The file is Priya Raman's call, `kit/network/priya-raman-2026-03-10.vcon.json`. Granted: `recording`, `transcription`, `fraud_investigation`, `customer_callback` (condition "phone only, no email"). Refused at `dialog[15]`: `marketing`, `model_training`, `quality_review`. `dialog[5]` holds the full card number and `dialog[7]` the last four of her Social Security number.

### Carrier
- Reads: `parties[n].tel`.
- Writes: the dialog. In production that is a `recording` entry with `start`, `duration`, `parties`, `mediatype` and the audio. Here the text dialog stands in for it. Covered by the `recording` grant.
- Never needs: `analysis`, `attachments`, the meaning of the words.

### Transcriber
- Reads: the dialog (the recording), its `start`, `duration` and `parties`. The dialog is the transcriber's input. In this file the dialog is already text, so for the role play treat it as the recording the carrier delivered. The transcript is still a new analysis entry, never an edit to the dialog.
- Writes: one `analysis` entry with `type: "transcript"`, `dialog` pointing at the source index, `vendor`, and the transcript in `body`. Speaker labels and word or segment timing go inside that body, in the vendor's format. Covered by the `transcription` grant.
- Never needs: party names, `tel`, the lawful basis body, other analysis. Writes nothing into `parties` or `dialog`.

### Summarizer
- Reads: the transcript analysis entry (here, the text dialog).
- Writes: an `analysis` entry with `type: "summary"`, `analysis` (index of the transcript it read) or `dialog`, and `vendor`. Allowed only for `fraud_investigation`. `model_training` is refused, so the call must not train its model.
- Never needs: the card number at `dialog[5]` or the SSN digits at `dialog[7]`. A redacted copy serves.

### CRM
- Reads: `parties[0].name` and `tel`, the summary once the Summarizer has written it (`analysis` starts empty), and the lawful basis grants.
- Writes: nothing required in the vCon. It may add a `tags` attachment.
- Never needs: raw dialog, the card number, the SSN digits. `marketing` is refused, so no card offers.

### Callback agent
- Reads: `parties[0].tel`, the summary once written (until then, `dialog[8]` to `dialog[12]`), and the `customer_callback` grant with its condition "phone only, no email", backed by `dialog[13]`.
- Writes: the callback is a new conversation and gets its own vCon.
- Never needs: the card number, the SSN digits, any email address. There is none in the file, and Priya refused email.

## Common mistakes

- Answering without opening the file. Fetch the raw URL. If that fails, say so. Never write a sample transcript and present it as the kit.
- Getting the transcriber backwards. The transcriber reads the dialog and writes the transcript. It does not receive a transcript as input.
- Putting diarization in `parties`. `parties` lists the people, once each. Who spoke a dialog entry is `dialog[n].parties`, an index. Speaker labels inside a transcript live in the transcript analysis body.
- Putting per-word or per-utterance timestamps in a recording's dialog entry. A recording entry has one `start` and `duration` for the whole recording. Finer timing lives in the transcript analysis body. The kit's text entries each carry a `start` because each one is a separate turn.
- Treating `role` or `originator` as standard fields. They are kit labels.
- Looking for tags or consent at the top level. Both are entries in `attachments[]`, found by `purpose`.
- Assuming permission when there is no lawful basis. The opener file has none on purpose. The right answer there is "the file does not say".
- Writing an analysis entry without `vendor`, or using `schema_version` instead of `schema`.

## Source

vCon core: https://datatracker.ietf.org/doc/draft-ietf-vcon-vcon-core/
Lawful basis extension: https://datatracker.ietf.org/doc/draft-howe-vcon-lawful-basis/
