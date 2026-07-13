# Run NetBSD in a Firecracker VM

This GitHub Action launches a Firecracker VM running NetBSD. The kernel boots in well under a
second, and the VM is typically reachable over ssh in a few seconds.

## Getting started

```yaml
- name: Launch Firecracker VM
  uses: acj/netbsd-firecracker-action@v0.1.0
  with:
    run-in-vm: |
      echo "Hello from inside the VM!"
```

## Supported inputs

### `pre-run`: Run commands after the VM starts

Runs **on the host** after the VM is up. The default rsyncs the workspace
into the VM. The `firecracker` ssh host alias is configured for you.

```yaml
- uses: acj/netbsd-firecracker-action@v0.1.0
  with:
    pre-run: |
      echo "Hello from outside the VM!"
    run-in-vm: ...
```

### `run-in-vm`: Run commands inside the VM

Runs **inside the VM** as root, with `sh -e`. Its exit code becomes the
step's exit code.

```yaml
- uses: acj/netbsd-firecracker-action@v0.1.0
  with:
    run-in-vm: |
      uname -a
      make && make test
```

### `post-run`: Run commands outside the VM before it shuts down

Runs **on the host** after `run-in-vm` exits (only on success, unless
`continue-on-error` is set). The default rsyncs the VM's `/root` back into
the workspace.

### `checkout`

Default: `true`

Check out the repository so the default `pre-run` can copy it into the VM.
Set to `false` if your workflow already ran `actions/checkout` or you don't
need the repo inside the VM.

### `continue-on-error`

Default: `false`

Run `post-run` (and copy artifacts out of the VM) even if `run-in-vm` failed.
The step still fails with the script's exit code.

### `disk-size`

Default: `2G`

Size to grow the VM's disk to, in units recognized by
[truncate(1)](https://linux.die.net/man/1/truncate). Sizes larger than the
shipped image trigger the first-boot resize reboot described above.

### `vcpu-count`

Default: `auto`

Number of vCPUs to expose to the VM. `auto` matches the host's logical core
count.

### `verbose`

Default: `false`

Print host diagnostics and stream the Firecracker log (which carries the
guest console) to the job log.

### `kernel-url`, `rootfs-url`, `firecracker-url`, `ssh-public-key-url`, `ssh-private-key-url`

Override where the boot artifacts come from. Defaults point at a
[netbsd-firecracker release](https://github.com/acj/netbsd-firecracker/releases).
Note that the published ssh keypair is, by definition, public — the VM is
only reachable from the runner via the tap device, but don't expose it to
anything else.
