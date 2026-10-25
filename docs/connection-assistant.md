# Connection assistant

Servers contains the assistant for the selected server. Diagnostics are on demand,
not part of snapshot polling. UI calls ServerSession / CommandRouter only.
Network operations return a job immediately so slow router discovery does not
block console commands or stop requests in the helper RPC queue.

## What is checked

- Current game port and bind address from server.properties / server settings.
- Local IPv4 TCP response (not a Minecraft protocol health check).
- Registered-server port collisions, and an available alternative in 25565-25664.
- Windows NATUPnP static TCP mappings, including destination, enabled state and
  the external IP reported by a mapping. No third-party IP service is contacted.
- External reachability is always **unverified**: a router mapping or NAT-loopback
  connection cannot establish internet reachability. An explicit link opens an
  external testing site; its result is not automatically imported into Flint.

New-server creation recommends an unoccupied, unregistered game port. Manual
port choices are preserved. The port is not reserved until the server binds it;
availability can change between creation and launch.

## Safety

UPnP mutation requires explicit consent. Only the current game TCP port can be
mapped, to an RFC1918 IPv4 address assigned to this PC. RCON and Bridge ports
are refused. Existing external-port entries are never deliberately overwritten.
Removal checks both a server-directory-specific Flint description and a local
destination address in the native operation. No other app's mappings are removed.

The Windows COM API creates static mappings: they can survive app/server exit.
Removal is explicit. DHCP or a router change may require manual cleanup in the
router. No automatic lease renewal, firewall changes, UAC elevation, router
password collection, public scanning or server restarts are performed.

PowerShell runs hidden with an encoded fixed program and a data-only JSON payload,
12-second timeout and bounded output. On timeout a mutation may have reached the
router; diagnose again before retrying. NATUPnP operations are not transactional
with competing router clients; always re-check after a concurrent router change.

UPnP unavailable does not imply internet hosting is impossible. UPnP might be
disabled, discovery might be blocked, or manual forwarding might already exist.
CGNAT, double NAT, IPv4-over-IPv6 service restrictions and firewall rules can
prevent external access despite a successful local mapping. IPv6 forwarding,
NAT-PMP/PCP, tunnels and automatic firewall configuration are not implemented.

Tests use isolated files, loopback sockets, fake gateway results, and parse the
PowerShell program without executing COM. Live router mapping and an external
connection must be verified by the user on their network.

## API references

- https://learn.microsoft.com/en-us/windows/win32/api/natupnp/nn-natupnp-istaticportmappingcollection
- https://learn.microsoft.com/en-us/windows/win32/api/natupnp/nf-natupnp-istaticportmappingcollection-add
- https://learn.microsoft.com/en-us/windows/win32/api/natupnp/nf-natupnp-istaticportmappingcollection-remove
