This is a small, Alpine-based version of the MiniDlna home media server meant to run in an extremely restricted docker container. It will serve media from a provided, read-only volume - for ease of use, a host folder can be mounted, or for increased security, a docker volume. As of now, host networking (net=host) is required in order to make auto-discovery work. User namespaces are also a goal - the container itself is compatible, but user namespaces are still incompatible with net=host.

Minidlna version: 1.2.0
Container version 1.1.4

Tested Security Configuration:
- Non-root UID/GID by default
- All capabilities dropped
- Cgroup resource limitation
- Sane ulimits
- Selinux isolation (Docker daemon must have selinux enabled)
- Read only root FS
- No new privileges flag enabled
- Tmpfs mounted database and PID directory
- Custom, more restrictive seccomp profile (link below) - reduces from ~315 to 64 system calls.

To be implemented:
- Drop need for net=host (pending working implementation of non net=host dependent auto-discovery)
- User namespaces (pending net=host workaround or compatibility)

No Longer Officially Supported:
- Grsecurity enabled kernel -- While compatible and historically hosted on a grsecurity and pax enabled host kernel, Grsecurity's decision to paywall the patches means that they are no longer available to most users without commercial backing. From 7/1/17 forward, configurations will be hosted on and tested against the latest official RHEL and CentOS kernels. With this in mind, it is assumed that you have no-new-privileges, seccomp, and selinux supported by your kernel.

(Lengthy) Command implementing these features (selinux must be enabled in your docker config for selinux protection):

docker run -d --restart=always --name=<container_name> --net=host -v <volume_path>:/opt:ro,z --read-only --tmpfs /run/db:rw,nosuid,nodev,noexec,uid=100000,gid=100000,mode=1770 --tmpfs /var/run/minidlna:rw,nosuid,nodev,noexec,uid=100000,gid=100000,mode=1770 --cap-drop=ALL -m 2G --memory-reservation 1G --cpus=0.8 --security-opt no-new-privileges  --security-opt seccomp=<path_to_seccomp_profile.json> --pids-limit 96 --ulimit nofile=2060:2560 --ulimit nproc=64:96 iotos/minidlna

Variables:
<container_name> = the name of the docker container
<volume_path> = the path to a media directory on the host or the name of a docker volume
<path_to_seccomp_profile.json> = the path on the host to the seccomp profile ending in .json

Seccomp profile:
With the loss of public Grsecurity, there is almost no other mechanism available to protect the kernel. Not even selinux can protect against many kernel exploits, which makes reducing the attack surface area more important than ever. The default docker profile reduces 315+ available system calls to around 265, with many of the dropped calls being added again based on set capabilities. The profile provided here is hand made, and reduces 315+ available system calls to 64, without the capability loophole. One caveat is that I have no way to test this profile with Arm/Arm64 hardware - compatibility is only guaranteed with i386/x86_64. The profile should be stored in a secure space on your FS, ideally only accessible by root.

Profile (compressed) download page: http://s000.tinyupload.com/?file_id=36954499112233723900
SHA256: 6ff948e86b16aa6d578fbeb6d4b2098caf12faeba8b7b68d3a8298e8532a8584
Scan results: https://www.virustotal.com/en/file/6ff948e86b16aa6d578fbeb6d4b2098caf12faeba8b7b68d3a8298e8532a8584/analysis/1502089112/
