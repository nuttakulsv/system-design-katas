# Prompt 07: Shared documents

## Prompt

"Design a collaborative document editor, where several people edit the same document at once."

## Hidden requirements

- Up to 50 simultaneous editors on one document; most documents have one to three.
- Works offline on mobile; edits made offline merge when back online.
- Full version history, with "restore to this version".
- Comments anchored to text that survive edits around them.
- Presence: see other people's cursors in real time.

## Numbers

- 10M documents, 500k editing sessions a day.
- Median document 20 KB; large ones up to 5 MB.
- Edit-to-visible latency for other editors: under 200 ms in the same region.

## A strong design covers

- **Conflict resolution:** CRDTs (for example a sequence CRDT like those in Yjs or Automerge) or operational transformation with a central server. A clear reason for the choice; offline editing pushes towards CRDTs.
- **Session server per document** (sticky routing by document ID) that relays operations over WebSockets, with presence kept in memory only.
- **Persistence:** an append-only operation log plus periodic snapshots; compaction; version history from snapshots and the log.
- **Anchors** for comments and cursors that are positions in the CRDT, not character offsets.
- **Permissions** checked on join and on every operation.
- **Scaling:** most documents are small and quiet; hot documents with 50 editors need batching of operations and backpressure.

## Probes

- "Two users delete and edit the same sentence offline, then reconnect. What do they see?"
- "The session server for a busy document crashes. What happens to unsaved operations?"
- "How does 'restore to version' work without breaking everyone's open sessions?"

## Curveballs

- "Legal needs to permanently erase a paragraph from history (personal data request)."
- "We want an AI assistant that edits the document alongside humans."

## Level notes

- **Senior:** picks OT or CRDT with a reason, a WebSocket relay, op log and snapshots.
- **Staff:** reasons through offline merge semantics, anchors, failure of the session server, and the cost of history; plans for hot documents.
- **Principal:** considers build versus adopting an existing CRDT library, data retention and erasure obligations, and how the editor becomes a platform (APIs, embedding, AI editing).

## Pitfalls

- Last-write-wins on the whole document.
- Character offsets for comments.
- Saving the full document on every keystroke.
