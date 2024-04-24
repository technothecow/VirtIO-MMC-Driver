FROM ghcr.io/matwey/docker/opensuse-kernel-deploy:latest

COPY dracut.conf.d/10-mmc.conf /usr/lib/dracut/dracut.conf.d

COPY dracut.conf.d/20-lspci.conf /usr/lib/dracut/dracut.conf.d
COPY modules.d/20lspci /usr/lib/dracut/modules.d/20lspci

COPY dracut.conf.d/30-dd.conf /usr/lib/dracut/dracut.conf.d
COPY modules.d/30dd /usr/lib/dracut/modules.d/30dd

COPY dracut.conf.d/31-hexdump.conf /usr/lib/dracut/dracut.conf.d
COPY modules.d/31hexdump /usr/lib/dracut/modules.d/31hexdump