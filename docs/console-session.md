# Console session history

Console history lasts for the Flint process lifetime, not a Minecraft process
lifetime. Rotating, truncating, deleting or temporarily losing `latest.log` does
not clear already collected entries. Each server log path has independent history.

- Core tails the selected server by byte offset in bounded chunks, including
  while the GUI is idle. Previously selected sources close their input handle;
  their collected history remains available when selected again. The first read
  imports the current `latest.log`, not older archives.
- An open file handle drains the previous file before attaching a rotated file.
- UTF-8 decoding carries partial characters and lines across reads.
- Entry IDs are stable and distinguish repeated identical log messages.
- Snapshots carry at most 1000 new entries / approximately 512 KiB per page, plus
  a cursor and backlog indicator. Exceptionally long lines are preserved as
  consecutive segments of at most 128 Ki characters.
- The client drains backlog promptly and merges by ID. Its live window is bounded
  to 5000 entries per server. Older entries remain in a private temporary JSONL
  spool, with sparse offsets, until Core exits. The Older logs action retrieves
  preceding pages without replacing the live cursor; Jump to latest resumes live
  viewing. Byte-limited backward pages are contiguous, including for long lines.
- The list is virtualized with measured variable-height rows. Older rows leave
  the DOM, not the history. Scrolling up disables following until Jump to latest.
- Searches and copying apply to the currently loaded live window or history page,
  not a whole-disk archive search. There is no 220-line truncation or deletion on
  Minecraft restart. Minecraft's existing log files are never changed by this.

History is a session disk spool, not a permanent archive. DOM size, live-window
memory and transfer size are bounded. Sparse index memory and temporary disk use
grow with recorded history. Spools are removed on graceful Core shutdown. An OS
crash or forced helper termination can leave a temporary spool and does not provide
automatic recovery into a new helper process.

## Command Responses

Manual commands, save-all responses and their failures are appended to the same
Core console stream, split into selectable lines with local receipt timestamps.
These entries survive server log rotation and are searchable/copyable like logs.
They are not written back to Minecraft's `latest.log`. In managed mode, command
dispatch acknowledgements are shown; actual server output still arrives via logs.

The ServerSession owns a command-only RCON channel. Consecutive commands reuse the
authenticated socket; no monitoring/heartbeat commands are sent. It closes after
two idle minutes, on server selection/configuration changes, stop, errors or helper
shutdown. An uncertain command is never automatically replayed. A later explicit
command can establish a new connection. Configuration changes are checked before
the next command, while server selection is also checked on snapshot refresh.

## Minecraft request framing

The Minecraft 1.20.1 RCON reader used by this Mohist build expects one complete
request in each read and disconnects if a read contains multiple requests. Flint
therefore waits for the first command response before sending its non-executing
response fence. Sending command and fence back-to-back can coalesce on TCP and
cause alternating successful operations and `RCON socket closed` failures.
The fence still collects all response chunks; quiet-time completion is not used.
No command is retried automatically. Commands exceeding the 1460-byte request
buffer (including framing), or containing null bytes, are refused before sending.
Idle cleanup waits until every queued command finishes. Closing a client also
cancels its queued commands instead of reconnecting to a previously selected server.

Console Tab stays in its input and completes a suggestion when one is available.
Normal page buttons are excluded from Tab cycling; text fields and dialog focus
traps retain keyboard navigation. Non-editable page surfaces suppress the browser
context menu without replacing inventory slot actions or native text editing menus.

## Bridge disconnect fix (0.3.1)

The reported stack trace occurs when a client disconnects during a response body
write. The previous catch block printed it and attempted a second HTTP response.
Expected IO disconnects are now handled quietly; an uncommitted snapshot failure
returns 503, but a committed response is never written again. Unexpected runtime
failures are still reported, rate-limited to one per minute.

The active Mohist profile had a 10 ms Bridge timeout, shorter than a normal server
tick. It is changed to 3000 ms. Core enforces a 500 ms runtime minimum, and saving
new out-of-range settings is rejected. This addresses the observed HTTP failures;
it does not establish or fix the cause of Minecraft crashes.

Deploy the new application and replace `plugins/FlintBridge.jar` with the 0.3.1
build only after a normal Minecraft shutdown. No hot plugin reload is performed.
