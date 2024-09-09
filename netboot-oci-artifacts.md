# Distributing network boot files via image manifest

Provisioning bootable containers on bare metal systems require OS installer and bootloader files to
be distributed along. This specification provides a standard way to encode and describe such
payload.

The main storage layout is described in [OCI
Artifacts](https://github.com/opencontainers/image-spec/blob/main/artifacts-guidance.md)
specification using an [OCI image
manifest](https://github.com/opencontainers/image-spec/blob/master/manifest.md) and stored in an OCI
registry.

All required files MUST be packaged and stored in the registry together in order to either boot into
container, or boot into installer which is responsible for deploying bootable image. The OCI image
manifest MUST NOT contain any associated configuration files required for booting.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" are to be interpreted as described in [RFC
2119](https://tools.ietf.org/html/rfc2119) (Bradner, S., "Key words for use in RFCs to Indicate
Requirement Levels", BCP 14, RFC 2119, March 1997).

The key words "unspecified", "undefined", and "implementation-defined" are to be interpreted as
described in the [rationale for the C99 standard][c99-unspecified].

An implementation is not compliant if it fails to satisfy one or more of the MUST, MUST NOT,
REQUIRED, SHALL, or SHALL NOT requirements for the protocols it implements. An implementation is
compliant if it satisfies all the MUST, MUST NOT, REQUIRED, SHALL, and SHALL NOT requirements for
the protocols it implements.

## Image config object

The configuration object MUST have the MIME type `application/vnd.oci.empty.v1+json`, and MUST be an
empty JSON object ('{}').  See the [Guidance for an empty
descriptor](https://github.com/opencontainers/image-spec/blob/main/manifest.md#guidance-for-an-empty-descriptor) 

Because the `config.mediaType` is set to the empty value, the `artifactType` MUST be defined and
currently it has MIME type `application/vnd.org.pulpproject.netboot.artifact.v1`.

## Image layers

Each layer in the manifest MUST map to exactly one Netboot file provided at build time. For example,
in order to boot Red Hat compatible x86_64 system via PXE, the following files are subject of
storing into layers:

* `shim.efi`
* `grubx64.efi`
* `pxelinux.0` or `grubx64.0`
* `vmlinuz`
* `initrd.img`
* `install.img`
* `boot.iso`

Each layer MUST have the MIME type of `application/x-netboot-file` and contain the following annotations:

* `org.opencontainers.image.title`: filename

## Image manifest tag naming conventions

An image manifest containing Netboot files MUST be tagged in the format of
`version-architecture` where, as an example:

* `fedora/fedora-netboot:40` is an image index for fc40
* `fedora/fedora-netboot:40-amd64` is an image manifest for fc40 arch amd64

Image manifest MUST have the following annotation `"org.pulpproject.netboot.version": "1"`

## Image index containing OCI artifacts

For the multi-arch builds an additional manifest MAY be created according to the [OCI image
index](https://github.com/opencontainers/image-spec/blob/main/image-index.md) specification.

Manifest index MUST have the following annotation `"org.pulpproject.netboot.version": "1"`

```
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.index.v1+json",
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:190ae282c4bc3b55dfd91239878c0c422898af8065244253d342f870f7c2263d",
      "size": 1541,
      "annotations": {
        "org.pulpproject.netboot.version": "1"
      },
      "platform": {
        "architecture": "amd64",
        "os": "linux",
        "os.version": "fedora-40"
      },
      "artifactType": "application/vnd.org.pulpproject.netboot.artifact.v1"
    },
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:6b26f794eafa6df574e41203b9542ae614436f00d7ee2f86e61bcbdc89a30ed1",
      "size": 1342,
      "annotations": {
        "org.pulpproject.netboot.version": "1"
      },
      "platform": {
        "architecture": "arm64",
        "os": "linux",
        "os.version": "fedora-40"
      },
      "artifactType": "application/vnd.org.pulpproject.netboot.artifact.v1"
    }
  ],
  "annotations": {
    "org.pulpproject.netboot.version": "1"
  }
}
```

## Example OCI Image Manifest containing PXE files stored as OCI artifacts

``` json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "application/vnd.org.pulpproject.netboot.artifact.v1",
  "config": {
    "mediaType": "application/vnd.oci.empty.v1+json",
    "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
    "size": 2,
    "data": "e30="
  },
  "layers": [
    {
      "mediaType": "application/x-netboot-file",
      "digest": "sha256:80c3fe2ae1062abf56456f52518bd670f9ec3917b7f85e152b347ac6b6faf880",
      "size": 196,
      "annotations": {
        "org.opencontainers.image.title": "boot.iso"
      }
    },
    {
      "mediaType": "application/x-netboot-file",
      "digest": "sha256:a3b7052d7b2f27ff73c677fde7e16e8664a2151f5bb0e6ade3a392c59f913557",
      "size": 3972416,
      "annotations": {
        "org.opencontainers.image.title": "grubx64.efi"
      }
    },
    {
      "mediaType": "application/x-netboot-file",
      "digest": "sha256:8ea1dd040e9725926ca2db5e30e0022f7c66c369b429b49336e7f95cdaf93ee7",
      "size": 149397724,
      "annotations": {
        "org.opencontainers.image.title": "initrd.img"
      }
    },
    {
      "mediaType": "application/x-netboot-file",
      "digest": "sha256:4723a6b6cf998f0571759569a18e390ba4be4df45dece95d0094a64f4e99a314",
      "size": 618008576,
      "annotations": {
        "org.opencontainers.image.title": "install.img"
      }
    },
    {
      "mediaType": "application/x-netboot-file",
      "digest": "sha256:fff4b2feeef35b3a03487a9d38e2f17b2dbd8241325c805a5aece86bcaf4f23a",
      "size": 42561,
      "annotations": {
        "org.opencontainers.image.title": "pxelinux.0"
      }
    },
    {
      "mediaType": "application/x-netboot-file",
      "digest": "sha256:4773d74d87c2371a25883b59a3b6d98d157de46933676706d215015b1130f2d1",
      "size": 949424,
      "annotations": {
        "org.opencontainers.image.title": "shimx64.efi"
      }
    },
    {
      "mediaType": "application/x-netboot-file",
      "digest": "sha256:09cf5df01619676e91e998fac6c456d67ec3cac25ee9244898b59699c588bb86",
      "size": 14966600,
      "annotations": {
        "org.opencontainers.image.title": "vmlinuz"
      }
    }
  ],
  "annotations": {
    "org.pulpproject.netboot.version": "1"
  }
}
```

[modeline]: # ( vim: set fenc=utf-8 tw=100 spell spl=en: )
