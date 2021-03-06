## cproxy

[![Continuous integration](https://github.com/NOBLES5E/cproxy/actions/workflows/build.yml/badge.svg)](https://github.com/NOBLES5E/cproxy/actions/workflows/build.yml)

`cproxy` can redirect TCP and UDP traffic made by a program to a proxy, without requiring the program supporting a proxy.

Compared to many existing complicated transparent proxy setup, `cproxy` usage is as easy as `proxychains`, but unlike `proxychains`, it works on any program (including static linked Go programs) and redirects DNS requests.

Note: The proxy used by `cproxy` should be a transparent proxy port (such as V2Ray's `dokodemo-door` inbound and shadowsocks `ss-redir`). A good news is that even if you only have a SOCKS5 or HTTP proxy, there are tools that can convert it to a transparent proxy for you (for example, [transocks](https://github.com/cybozu-go/transocks), [ipt2socks](https://github.com/zfl9/ipt2socks) and [ip2socks-go](https://github.com/lcdbin/ip2socks-go)).

## Installation

You can install by downloading the binary from the [release page](https://github.com/NOBLES5E/cproxy/releases) or install with `cargo`:

```
cargo install cproxy
```

## Usage

### Simple usage: just like `proxychains`

You can launch a new program with `cproxy` with:

```
cproxy --port <destination-local-port> -- <your-program> --arg1 --arg2 ...
```

All TCP connections and DNS requests will be proxied. In this case, your local transparent proxy should support DNS address overriding to make DNS requests redirection work properly. For an example setup, see [wiki](https://github.com/NOBLES5E/cproxy/wiki/Example-setup-with-V2Ray). If you don't want to proxy DNS requests, run with

```
cproxy --port <destination-local-port> --no-dns -- <your-program> --arg1 --arg2 ...
```

### Simple usage: use iptables tproxy

If your system support `tproxy`, you can use `tproxy` with `--use-tproxy` flag:

```bash
cproxy --port <destination-local-port> --use-tproxy -- <your-program> --arg1 --arg2 ...
# or for existing process
cproxy --port <destination-local-port> --use-tproxy --pid <existing-process-pid>
```

With `--use-tproxy`, there are several differences:

* All UDP traffic are proxied instead of only DNS UDP traffic to port 53.
* Your V2Ray or shadowsocks service should have `tproxy` enabled on the inbound port. For V2Ray, you need `"tproxy": "tproxy"` as in [V2Ray Documentation](https://www.v2ray.com/en/configuration/transport.html#sockoptobject). For shadowsocks, you need `-u` as shown in [shadowsocks manpage](http://manpages.org/ss-redir).

An example setup can be found [here](https://github.com/NOBLES5E/cproxy/wiki/Example-setup-with-V2Ray).

### Advanced usage: proxy an existing process

With `cproxy`, you can even proxy an existing process. This is very handy when you want to proxy existing system services such as `docker`. To do this, just run

```
cproxy --port <destination-local-port> --pid <existing-process-pid>
```

The target process will be proxied as long as this `cproxy` command is running. You can press Ctrl-C to stop proxying.

## How does it work?

With the help of linux `cgroup`, the implementation is very simple! Just read through https://github.com/NOBLES5E/cproxy/blob/master/src/main.rs :)

## Limitations

* `cproxy` requires `sudo` and root access to modify `cgroup`.
* Currently only tested on Linux.

## Similar projects

There are some awesome existing work:

* [graftcp](https://github.com/hmgle/graftcp): work on most programs, but cannot proxy UDP (such as DNS) requests. `graftcp` also has performance hit on the underlying program, since it uses `ptrace`.
* [proxychains](https://github.com/haad/proxychains): easy to use, but not working on static linked programs (such as Go programs).
* [proxychains-ng](https://github.com/rofl0r/proxychains-ng): similar to proxychains.
