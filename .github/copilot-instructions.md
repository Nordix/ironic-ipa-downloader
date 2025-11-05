# Ironic IPA Downloader - AI Coding Assistant Instructions

## Project Overview

Simple utility container that downloads Ironic Python Agent (IPA)
kernel and initramfs images from configured URLs and places them in a
shared volume for use by Ironic services. Used as an init container in
BMO deployments.

## Architecture

Single-purpose init container:

1. Downloads IPA kernel from `IPA_KERNEL_URL`
2. Downloads IPA initramfs from `IPA_INITRAMFS_URL`
3. Saves to `/shared/html/images/` directory
4. Exits after download complete

## Key Files

- `Dockerfile` - Minimal image with curl/wget
- `Makefile` - Build automation
- Download script (typically inline in Dockerfile ENTRYPOINT)

## Build Process

```bash
# Build image
make build

# Or directly
podman build -t quay.io/metal3-io/ironic-ipa-downloader:latest .
```

## Usage

Deployed as init container in Ironic pod:

```yaml
initContainers:
- name: ironic-ipa-downloader
  image: quay.io/metal3-io/ironic-ipa-downloader
  env:
  - name: IPA_KERNEL_URL
    value: "https://images.rdoproject.org/.../ironic-python-agent.kernel"
  - name: IPA_INITRAMFS_URL
    value: "https://images.rdoproject.org/.../ironic-python-agent.initramfs"
  volumeMounts:
  - name: shared
    mountPath: /shared
```

## Configuration

Environment variables:

- `IPA_KERNEL_URL` - URL to IPA kernel image
- `IPA_INITRAMFS_URL` - URL to IPA initramfs image
- `IPA_KERNEL_FILE` - Target filename
  (default: `ironic-python-agent.kernel`)
- `IPA_INITRAMFS_FILE` - Target filename
  (default: `ironic-python-agent.initramfs`)

## Integration

- Used by BMO's ironic-deployment manifests
- Runs before Ironic container starts
- Ensures IPA images available before provisioning begins
- Shares `/shared` volume with Ironic, dnsmasq, httpd containers

## Common Pitfalls

1. **Download Failures** - Check URL accessibility from cluster
2. **Permissions** - Ensure init container can write to shared volume
3. **Image Compatibility** - IPA version must match Ironic version
4. **Storage Space** - Shared volume must have enough space for images

Simple utility but critical for Metal3 deployment - no IPA images means no provisioning.
