# Xrust protocol profile

Supported release platforms: OpenWrt aarch64 only.
CI tests run on Linux; both release targets use static musl binaries.
Other platform code inherited from upstream is not a supported build target.

Firefox/Safari and Chrome for XHTTP now use our Sing-Box X 1.0.3 captures.
See [the uTLS refresh report](RUST-X-UTLS-2026-10-10.md).

Xrust does not ship Naive, Shadowsocks, TUIC, AnyTLS, Snell,
ShadowTLS, FPTN, outbound SOCKS, OpenVPN or a Trojan server (inbound). Config
`check` rejects these types. SOCKS4/4a client requests are rejected; SOCKS5
and HTTP mixed inbounds remain available for Forkop DirectProxy.

VLESS/Vision, the Trojan client (outbound), TLS/uTLS/REALITY, WebSocket,
HTTPUpgrade, HTTP/2, gRPC, XHTTP, Hysteria2, WireGuard/AmneziaWG, MASQUE,
Tailscale, DNS, SRS, TProxy, Redirect, TUN, direct, selectors/urltest and
Clash API remain enabled.

The Trojan outbound takes the same `tls` and `transport` options as VLESS and
carries TCP and UDP. It requires TLS, and `multiplex` is refused at `check`.
The version block lists it as `outbound.trojan` in its `Features:` line, which
is what Forkop X looks for before it generates Trojan outbounds. The request
head goes out in the same write as the first payload, as in both reference
clients; a destination that speaks first gets it alone after 100 ms, as with
Xray.

Release build: `cargo build --locked -p core-cli --no-default-features --features rust-x --release --target x86_64-unknown-linux-musl`.
There is a single protocol profile: the default features build the same core
as the release command. `rust-x` stays as an alias of `full`, so packaging
scripts that pass `--no-default-features --features rust-x` keep working.
Profile tests: `cargo test -p core-config --test rust_x_profile`.

The removed protocols used to sit behind an optional `legacy-protocols` Cargo
feature; that feature and the modules only it compiled were deleted, and can
be recovered from git history if needed: the last tree with them is the
parent of commit `af793ad`. Keep fork-specific fixes in focused
commits and port upstream changes without replacing our DNS, routing,
monitoring or OpenWrt behavior; upstream protocols outside this profile are
not imported.

Shared helpers under VMess/Shadowsocks/FPTN remain in the source because
retained protocols use them. Release LTO removes unreachable implementations.
OpenWrt APK/IPK packaging and installation checks are described in
[OpenWrt packages](OPENWRT-PACKAGES.md).
