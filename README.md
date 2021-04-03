# gateway_jumper

An Expect script used to automate the otherwise interactive process of entering
passwords and a TOTP code, when setting up a sshuttle connection to a
destination host via a jumphost, if no other authentication options are
available.

**Example of command and prompts which will be filled in for you:**

```
sshuttle -r <destination_host>

[local sudo] Password: <GWJ_LOCAL_PASS>
Password: <GWJ_JUMP_PASS>
Enter Your Microsoft verification code <totp_response>
Password: <GWJ_DEST_PASS>
```

> :warning: Passwords need to be defined in plain text in their corresponding
            environment variables, so be a little bit careful with this.



# Requirements

You will need to have [Expect][1], [oathtool][2] and [sshuttle][3] installed in
order for this script to work. Here is a one-liner to install them on Debian 10
Buster:

```bash
apt install expect oathtool sshuttle
```



# Usage

In its properly configured form this script is invoked like this:

```bash
./gateway_jumper <destination_host>
```

where `<destination_host>` is the address of the host machine you want to
connect to. It will "spawn" a sshuttle process which tries to connect to the
address given, and it expects that this connection goes through a jumphost which
require a password and a TOTP code as authentication. However, there are some
setup steps which needs to be completed before this is possible to fully
automate.

### Configure SSH
For the easiest use of this script I suggest you populate your `~/.ssh/config`
with something like this:

```
host jump_host
 hostname jumphost.org
 user username

host destination_host
 hostname destination.org
 user username
 proxycommand ssh jump_host -W %h:%p

host *
 ForwardX11 no
 Compression yes
```

Here you need to change at least the `username`, `jumphost.org` and
`destination.org` directives to what is correct for you. The `jump_host` and
`destination_host` names can also be changed to something that you fancy better.

### Configure this Script
At the top of the [`gateway_jumper`](./gateway_jumper) script there are four
variables which needs to be properly configured. This can either be done by
setting the environment variables mentioned below, or by just replacing the
`$env(*)` reader functions inside the file with the correct values directly.

1. `GWJ_LOCAL_PASS`\
   The password for invoking sudo on the local computer.\
   Set to empty string (`""`) if no password is needed for local sudo.
2. `GWJ_JUMP_PASS`\
   This need to be set to a non-empty string, since the jumphost should require
   a password.
3. `GWJ_TOTP_SECRET`\
    This is the Base32 secret key used to create the TOTP challenge response
    code. \
    Example: "ABCDE12FG3HIJ45K"
4. `GWJ_DEST_PASS`\
    The password on the final destination host.\
    Set to empty string (`""`) if no password is needed (e.g. key-based
    authentication is used).

### Test the Connection
Before running this automatic script I **strongly** suggest manually connecting
to the destination host at least once so you know that the SSH command actually
works.

```bash
ssh <destination_host>
```

If you the destination host has password authentication you should run into
the following prompts:

```bash
Password:
Enter Your Microsoft verification code
Password:
```

Make sure what you type into these correspond to what the variables `2`, `3`
and `4` in the list [above](#configure-this-script) would have returned. To
generate a TOTP response you can run the following command:

```bash
oathtool --base32 --totp "${GWJ_TOTP_SECRET}"
```

### (Optional) Edit the `sshuttle` Command
You can do quite a lot with [sshuttle][3], so if you have any personal settings
you want to use it should be easy to add this on the
["spawn" line](./gateway_jumper#L26) in the script.

You are also not limited to only running `sshuttle`, you could change the
command after the "spawn" to any command which would cause the same prompts
as mentioned [in the beginning](#gateway_jumper) to appear and still use this
script. Or you can use this script as a start to automate you own special case.

### (Optional) Add this Script to `$PATH`
If you want to be able to call this script from anywhere on your system you can
add this folder to your `$PATH`. This can be done by including the following
line at the bottom of your `~/.bashrc` or `~/.zshrc` file:

```
PATH="/path/to/gateway_jumper:${PATH}"
```

By sourcing the edited file again, or just opening a new terminal, it should
now be possible to use `gateway_jumper` without having to provide the full path.






[1]: https://linux.die.net/man/1/expect
[2]: https://manpages.ubuntu.com/manpages/trusty/man1/oathtool.1.html
[3]: https://sshuttle.readthedocs.io/en/stable/manpage.html
