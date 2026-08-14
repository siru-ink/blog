+++
title = "Using SSH Keys for VPS Login"
date = "2025-12-31"
+++
This is a simple guide on how to get passwordless logins over
[ssh](https://en.wikipedia.org/wiki/Secure_Shell) to a server up and running on
a debian linux based system. By no means is this guide authoritative. It is more
of a reminder for my future self on how I did this the next time I get a new
[VPS](https://en.wikipedia.org/wiki/Virtual_private_server) up and running
again.

## Generating a Local SSH Key

First, there must exist a local ssh key. On most unix/linux based systems, per
user ssh information is stored in the user's home directory (`~`) in a dedicated
`.ssh` subdirectory. If there is a key, such as `id_ed25519` and a corresponding
`id_ed25519.pub`, then this computer already has a private and public ssh key
which can be used in the next steps. If not, the following command can be used
to generate a new key:

```bash
ssh-keygen
```

This command first prompts for a location to store the new key and a filename.
I would highly recommend sticking with the default here, unless there is a
particular reason not to. Second, it asks for a password for the key. This
password can be skipped by simply leaving it empty.[^1]

## Uploading the SSH Key to the Server

Once a private/public key pair[^2] exist, the public version of this key needs
to be registered on the server as an allowed login key. The easiest way to do
this is to use the `ssh-copy-id` tool as:

```bash
ssh-copy-id -i ~/.ssh/<new-key-name>.pub <user>@<server-ip-or-domain>
```

With the substitutions being:

- `<new-key-name>` being the name of the ssh key. If left with the default name,
  this would most likely be `id_ed25519`
- `<user>` being the user on the server for whom passwordless login is being
  created
- `<server-ip-or-domain>` being the normal ssh access address for the server

> [!Note]
> This command will ask you for your ssh login password to be able to modify the
> ssh configuration for the user on the server.

### What `ssh-copy-id` Does

The above command makes a copy of the local public (`.pub`) version of the ssh
key. Then, it logs into the server and appends this ssh key to the end of the
`authorized_keys` configuration file.[^3] Both of these steps can also be done
manually if that is preferred. The only caveat being to make sure not to delete
any other ssh keys already previousely registered in the `authorized_keys`
configuration file.

## Ensure that PubkeyAuthentication is Enabled on the Server

Lastly, it might be worth while to double check that public key authentication
is activated for the ssh daemon on the server. By default Debian linux should
have this enabled, but the configuration can be double checked in the
`/etc/ssh/sshd_config` file. There should be a line reading:

```
PubkeyAuthentication yes
```

> [!Note]
> It is important to make a destinction between the `ssh_config` and
> `sshd_config` files here. the above mentioned config file is `sshd_config`. As
> a general rule of thumb, ssh is relevant for outbound connections and sshd
> refers to _ssh daemon_ for inbound connections. So login settings will always
> be found in the _sshd_ files.

## Login with SSH Private Key

Lastly, this private ssh key must be referenced in the ssh login command, as:

```bash
ssh -i ~/.ssh/<private-key-name> <user>@<server-ip-or-domain>
```

With:

- `<private-key-name>` being the name of the private key (no `.pub` extention)
  that is the pair of the public key copied to the server
- `<user>` being the user on the server for whom this passwordless login was set
  up
- `<server-ip-or-domain>` being the server address such as *www.example.com*.

This command can also be aliased using something akin to this in whatever shell
is being used on the local computer:

```bash
alias sshconnect="ssh -i ~/.ssh/<private-key-name> <user>@<server-ip-or-domain>
```

[^1]: Theoretically, it might be slightly more secure if the ssh private key
      were to be encrypted with a password, however, if the base premise of
      using this ssh key is to create a passwordless login, then setting a
      password here would defeat the point.

[^2]: The private and public key pair can be differentiated by the `.pub`
      extention on the public key. Only the public key should ever be shared off
      of the computer where it was created to maintain security.

[^3]: The per user `authorized_keys` configuration file can normally be found in
      `~/.ssh/authorized_keys`.
