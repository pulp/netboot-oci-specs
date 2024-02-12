# Packaging Network Boot files using Image manifest

Netboot files (e.g. PXE) can be packaged as [OCI Artifacts](https://github.com/opencontainers/image-spec/blob/main/artifacts-guidance.md)
using an [OCI image manifest](https://github.com/opencontainers/image-spec/blob/master/manifest.md)
and stored in OCI registry.

All required files MUST be packaged and stored in the registry together in order to either boot into container,
or boot into installer which is responsible for deploying bootable image.
The OCI image manifest MUST NOT contain any associated configuration files required for booting.


## Image config object

The configuration object MUST have the MIME type `application/vnd.oci.empty.v1+json`, and MUST be
an empty JSON object ('{}').
See the [Guidance for an empty descriptor](https://github.com/opencontainers/image-spec/blob/main/manifest.md#guidance-for-an-empty-descriptor) 

Because the `config.mediaType` is set to the empty value, the `artifactType` MUST be defined and
currently it has MIME type `application/vnd.unknown.artifact.v1`.


## Image layers

Each layer in the manifest MUST map to one Netboot file provided at build time.

For example, in order to boot Red Hat compatible system via PXE, the following files are subject of storing into layers:

* install.img
* vmlinuz
* initrd.img
* shim.efi
* grubx64.efi

Where each layer MUST have:

* the MIME type `application/x-netboot-file` or `application/x-netboot-file+zstd` if compressed
* the annotations `org.opencontainers.image.description` and `org.opencontainers.image.title`
  describing each file

For the rest of image definition refer to the [OCI specs](https://github.com/opencontainers/image-spec).


## Example OCI Image Manifest containing PXE files stored as OCI artifacts

``` json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "application/vnd.unknown.artifact.v1",
  "config": {
    "mediaType": "application/vnd.oci.empty.v1+json",
    "digest": "sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "size": 0,
    "annotations": {
      "org.opencontainers.image.description": "Anaconda installer bootloader and Linux kernel/initrd. Usage: oras pull ghcr.io/lzap/bootc-netboot-example:TAG"
    }
  },
  "layers": [
    {
      "mediaType": "application/x-netboot-file",
      "digest": "sha256:0d7a9a3c4804334b23cd43ffc3aedad4620192d9c520e2f466f56b96aeb2a284",
      "size": 13335480,
      "annotations": {
        "org.opencontainers.image.description": "Anaconda installer Linux kernel",
        "org.opencontainers.image.title": "vmlinuz"
      }
    },
    {
      "mediaType": "application/x-netboot-file",
      "digest": "sha256:4080a4d952d5145625d18b822214982a87ad981c254fcac671ca9ea245da5e3d",
      "size": 102417772,
      "annotations": {
        "org.opencontainers.image.description": "Anaconda installer init RAM disk",
        "org.opencontainers.image.title": "initrd.img"
      }
    },
    {
      "mediaType": "application/x-netboot-file",
      "digest": "sha256:32e77976ebbc915f77dd7f15d66a52cb177d5a9d2ee1794b173390b67495c047",
      "size": 946736,
      "annotations": {
        "org.opencontainers.image.description": "SecureBoot shim EFI binary",
        "org.opencontainers.image.title": "shim.efi"
      }
    },
    {
      "mediaType": "application/x-netboot-file",
      "digest": "sha256:735284626212a6267c0e90dab2428e8f82c182af17aec567c80838d219d9fa42",
      "size": 2532984,
      "annotations": {
        "org.opencontainers.image.description": "Grub bootloader binary",
        "org.opencontainers.image.title": "grubx64.efi"
      }
    }
  ],
  "annotations": {
    "org.opencontainers.image.created": "2024-02-06T17:18:04Z",
    "org.opencontainers.image.description": "Anaconda installer bootloader and Linux kernel/initrd. Usage: oras pull ghcr.io/lzap/bootc-netboot-example:TAG"
  }
}
```


## Image tag naming conventions

An image manifest containing Netboot files MUST be tagged in the format of ``version-architecture`` where:

* ``version`` SHOULD specify OS version number in lowercase alphanumeric with dots or underscores without
  dash character (e.g. 39 or 2024.04 or 9.2).
  For rolling-release operating systems, version SHOULD contain date or timestamp, sortable order
  is preferred.
* ``architecture`` SHOULD be one of the values listed in the Go Language document for
  [GOARCH](https://go.dev/doc/install/source#environment).


## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" are to be interpreted as described in
[RFC 2119](https://tools.ietf.org/html/rfc2119) (Bradner, S., "Key words for use in RFCs to
Indicate Requirement Levels", BCP 14, RFC 2119, March 1997).

The key words "unspecified", "undefined", and "implementation-defined" are to be interpreted as
described in the [rationale for the C99 standard][c99-unspecified].

An implementation is not compliant if it fails to satisfy one or more of the MUST, MUST NOT,
REQUIRED, SHALL, or SHALL NOT requirements for the protocols it implements.
An implementation is compliant if it satisfies all the MUST, MUST NOT, REQUIRED, SHALL, and
SHALL NOT requirements for the protocols it implements.
