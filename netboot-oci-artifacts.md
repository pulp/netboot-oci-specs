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
currently it has MIME type `application/vnd.unknown.artifact.v1`.

## Image layers

Each layer in the manifest MUST map to exactly one Netboot file provided at build time. For example,
in order to boot Red Hat compatible x86_64 system via PXE, the following files are subject of
storing into layers:

* `shim.efi`
* `grubx64.efi`
* `vmlinuz`
* `initrd.img`
* `install.img`

Each file MUST be compressed via zstd compression algorithm. Each layer MUST have the MIME type of
`application/x-netboot-file+zstd` and contain the following annotations:

* `org.opencontainers.image.title`: filename
* `org.pulpproject.netboot.src.digest`: sha256 sum of the uncompressed file contents
* `org.pulpproject.netboot.src.size"`: uncompressed file size

## Image manifest tag naming conventions

An image manifest MUST contain the following annotations:

* `org.pulpproject.netboot.os.arch`: SHOULD be one of the values listed in the Go Language document
  for [GOARCH](https://go.dev/doc/install/source#environment)

* `org.pulpproject.netboot.os.name`: SHOULD be OS lowercase name (e.g `rhel`, `fedora`)

* `org.pulpproject.netboot.os.version`: SHOULD specify OS version number in lowercase alphanumeric
  with dots or underscores without dash character (e.g. `39` or `2024.04` or `9.2`). For
  rolling-release operating systems, version SHOULD contain date or timestamp.

* `org.pulpproject.netboot.entrypoint`: filename to be loaded in order to initiate installation
  (bootloader or shim)

* `org.pulpproject.netboot.altentrypoint`: alternative filename to be loaded

* `org.pulpproject.netboot.legacyentrypoint`: legacy filename to be loaded (e.g. BIOS on x86_64
  platform)

An image manifest containing Netboot files MUST be tagged in the format of
`name-version-architecture` where

* `name` matches the `org.pulpproject.netboot.os.name` defined above
* `version` matches the `org.pulpproject.netboot.os.version` defined above
* `architecture` matches the `org.pulpproject.netboot.os.arch` defined above

## Image index containing OCI artifacts

For the multi-arch builds an additional manifest MAY be created according to the [OCI image
index](https://github.com/opencontainers/image-spec/blob/main/image-index.md) specification.

```
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.index.v1+json",
  "artifactType": "application/vnd.unknown.artifact.v1",
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:93a5ef6e62b75db1fd6c95a4e164b02669d569b23aab20d1dccbb0c381111e35",
      "size": 507,
      "annotations": {
        "netboot": "pxe"
      },
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      }
    },
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:858f6fd30a0336eeb615918dae8cdc091874fb560826f106b86a80ed12140775",
      "size": 508,
      "annotations": {
        "netboot": "pxe"
      },
      "platform": {
        "architecture": "aarch64",
        "os": "linux"
      }
    }
  ]
}
```

## Example OCI Image Manifest containing PXE files stored as OCI artifacts

``` json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "application/vnd.unknown.artifact.v1",
  "config": {
    "mediaType": "application/vnd.oci.empty.v1+json",
    "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
    "size": 2,
    "data": "e30="
  },
  "layers": [
    {
      "mediaType": "application/x-netboot-file+zstd",
      "digest": "sha256:8c4db4474646a08e4a251e2c1055cf5cf2c1c21f159e9a3ba74a381414652ad9",
      "size": 377793,
      "annotations": {
        "org.opencontainers.image.title": "shim.efi",
        "org.pulpproject.netboot.src.digest": "sha256:32e77976ebbc915f77dd7f15d66a52cb177d5a9d2ee1794b173390b67495c047",
        "org.pulpproject.netboot.src.size": "946736"
      }
    },
    {
      "mediaType": "application/x-netboot-file+zstd",
      "digest": "sha256:8526a40f8b5aa92dd35b01c03f2e748912d56b84c8675fe2f21e36a39b8eb388",
      "size": 565332,
      "annotations": {
        "org.opencontainers.image.title": "grubx64.efi",
        "org.pulpproject.netboot.src.digest": "sha256:735284626212a6267c0e90dab2428e8f82c182af17aec567c80838d219d9fa42",
        "org.pulpproject.netboot.src.size": "2532984"
      }
    },
    {
      "mediaType": "application/x-netboot-file+zstd",
      "digest": "sha256:d50baa5d4bf3af0fabebc7871b884be83f368c83dbe0ce1733087615808a6c15",
      "size": 41637,
      "annotations": {
        "org.opencontainers.image.title": "pxelinux.0",
        "org.pulpproject.netboot.src.digest": "sha256:dfcdf626efa753db88de0bf513c4c2e1c4e46cf084371e294e5f6864f16c2e01",
        "org.pulpproject.netboot.src.size": "42555"
      }
    },
    {
      "mediaType": "application/x-netboot-file+zstd",
      "digest": "sha256:e0180821662f2072771ecfbe4a242b3d7782126d06e1fa965f61e1247485943d",
      "size": 13020437,
      "annotations": {
        "org.opencontainers.image.title": "vmlinuz",
        "org.pulpproject.netboot.src.digest": "sha256:0d7a9a3c4804334b23cd43ffc3aedad4620192d9c520e2f466f56b96aeb2a284",
        "org.pulpproject.netboot.src.size": "13335480"
      }
    },
    {
      "mediaType": "application/x-netboot-file+zstd",
      "digest": "sha256:98328929370bdce755daaba59003399e5554a169feaf3cd9734c3f398f2f5b1c",
      "size": 100870099,
      "annotations": {
        "org.opencontainers.image.title": "initrd.img",
        "org.pulpproject.netboot.src.digest": "sha256:4080a4d952d5145625d18b822214982a87ad981c254fcac671ca9ea245da5e3d",
        "org.pulpproject.netboot.src.size": "102417772"
      }
    }
  ],
  "annotations": {
    "org.pulpproject.netboot.entrypoint": "shim.efi",
    "org.pulpproject.netboot.altentrypoint": "grubx64.efi",
    "org.pulpproject.netboot.legacyentrypoint": "pxelinux.0",
    "org.pulpproject.netboot.os.arch": "x86_64",
    "org.pulpproject.netboot.os.name": "rhel",
    "org.pulpproject.netboot.os.version": "9.3.0"
  }
}
```

[modeline]: # ( vim: set fenc=utf-8 tw=100 spell spl=en: )
