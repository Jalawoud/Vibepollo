# Bazzite port checkpoint

Target supplied by the tester: Bazzite NVIDIA Open 44.20261026, KDE Wayland,
RTX 5080, booted kernel 7.2.11-ogc4.1.fc44.x86_64, matching kernel-devel,
Secure Boot enabled, SELinux enforcing. Artemis is the client. Managed virtual
displays are required. Post-login-only streaming is acceptable initially.

Implemented here: an offline, compile-only driver qualification bundle and
behavioral tests. This is not an installable port. Both pinned drivers compiled successfully on the tester's 7.2.11 kernel.
Their vermagic matches that running kernel. Neither has been signed or loaded.
The logs report missing pahole/vmlinux BTF metadata; compilation still succeeded.

## Source findings

- `cmake/packaging/linux.cmake` supplies RPM dependency and privileged file
  ownership/capability metadata. The CPack lifecycle hooks still execute live
  setup, permissions changes and service control.
- `packaging/linux/copr/Sunshine.spec` has a separate partial rpm-ostree branch;
  substantial work occurs before that branch. It is not proof of working
  Bazzite installation and must not be used as an unattended installer as-is.
- The pinned libvirtualdisplay direct installer targets `/lib/modules` and uses
  depmod. DKMS also assumes a mutable module installation tree. Source compilation
  is already staged in a temporary directory; module delivery and update handling
  need a separate immutable-host design.
- The virtual-display driver supports kernel versions >=6.16 by policy; that
  alone does not establish API compatibility with the supplied 7.2.11 kernel.
- `build-module` selects LLVM when the target kernel config says Clang.
- The KWin GPU bridge and its native hooks need compilation against Bazzite's
  actual KWin development ABI, and must be tested with the NVIDIA driver.

## Next gates

1. Compile both pinned kernel drivers against the tester's kernel. Obtain logs,
   resolve compiler or kernel API failures, then freeze the successful source.
2. Build host + bridge in a Fedora 44 build environment; test the RPM payload
   and every lifecycle script in an offline deployment composition.
3. Deliver signed modules through a kernel-specific RPM or image build. Do not
   unlock /usr or disable signature enforcement. Enrollment is an explicit local
   hardware test; compilation alone cannot validate trust or module loading.
4. Move machine profile configuration to an explicit post-reboot step and keep
   system services inactive during composition. Preserve existing lifecycle
   safety checks for live hosts; never blanket-skip them based on the presence
   of the rpm-ostree executable.
5. Exercise SELinux enforcing, permissions, KWin bridge, KMS capture, NVENC,
   audio/input, virtual-display cleanup, reconnect, and reboot on the real host.
6. Qualify an OS/kernel update and rollback. A stale module must fail clearly,
   without replacing the user's working physical display or disabling security.

GitHub account verified through the connector: Jalawoud. Fork now exists at
Jalawoud/Vibepollo. The bazzite-port branch contains this checkpoint and a Fedora
44 artifact-only host build workflow. That workflow does not install RPMs, sign
modules, publish releases, or establish a working streaming session.
