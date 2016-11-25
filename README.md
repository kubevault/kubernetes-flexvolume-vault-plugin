# What is this ?

This project is an implementation of the flexvolume kubernetes plugin to inject a scoped vault token inside pods at startup so they can get their secrets.

# How do I build it ?

Just run `make` ( or ` go build -o whatever .` where `whatever` is the name you want the binary to have )
All dependencies are versionned under `/vendor` with glide and commited.

You can also `go get github.com/fcantournet/kubernetes-flexvolume-vault-plugin`

# How does it work ?

It creates a tmpfs volume and mounts it at a path specify by the kubelet.
Inside the volume are 2 files with a configurable _basename_:
    `basename` that contains the raw wrapped vault token.
    `basename.json` that contains the full response from vault at token creation time (includes metadata)

The token is scoped to a policy defined by a parameter provided to the plugin via stdin by the kubelet (cf. flexvolume documentation)

The binary generated by the project must be present on the node in a directory specified to the kubelet by the flag `--volume-plugin-dir`

it expects a vault token at a configurable path (set by `VAULTTMPFS_GENERATOR_TOKEN_PATH`) with a policy that allows the creation of token


# Configuration

Since the kubelet runs the plugin with a fixed set of arguments we can't pass configuration via flags in the command line.
We therefore use environment variables. The process inherits all the environment from the kubelet.

The plugin supports some the standard `vault` environment variables [as defined here](https://www.vaultproject.io/docs/commands/environment.html) (it calls `config.ReadEnvironment()`)
This means that all the defaults for these are set by Vault and the default value specified in the table below are subject to being FALSE
 (althought you should probably never use default values)
Vault loads system's CAs by default, but you can specifiy a custom CA certificate with `VAULT_CACERT` or `VAULT_CAPATH`.

Additionally we have variables to configure settings external to vault. These are prefixed with `VAULTTMPFS_` so as to not conflict with anything else.

(non-exhaustive) Table of supported configuration variables :

| Environment Variable              | default                    | Description                                                                                                                                   |
|-----------------------------------|----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| `VAULTTMPFS_GENERATOR_TOKEN_PATH` | /etc/kubernetes/vaulttoken | The path to load the token used by this service from.                                                                                         |
| `VAULTTMPFS_TOKEN_FILENAME`       | vault-token                | The name of the file in the created volume that will contain the wrapped token                                                                |
| `VAULT_ADDR`                      | https://127.0.0.1:8200     | The vault server URL                                                                                                                          |
| `VAULT_TLS_SERVER_NAME`           | ""                         | If set, use the given name as the SNI host when connecting via TLS.                                                                           |
| `VAULT_WRAPP_TTL`                 | 5m                         | TTL of the wrapped Token inserted in the volume.                                                                                              |
| `VAULT_MAX_RETRY`                 | 2                          | The maximum number of retries when a 5xx error code is encountered. Default is 2, for three total tries; set to 0 or less to disable retrying |
