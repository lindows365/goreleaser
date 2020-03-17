---
title: HTTP Upload
series: customization
hideFromIndex: true
weight: 120
---

GoReleaser supports building and pushing artifacts to HTTP servers using simple
HTTP requests.

## How it works

You can declare multiple `uploads` instances. All binaries generated by your
`builds` section will be pushed to each configured upload.

If you have only one `uploads` instance, the configuration is as easy as adding
the upload target and a username to your `.goreleaser.yml` file:

```yaml
uploads:
  - name: production
    target: http://some.server/some/path/example-repo-local/{{ .ProjectName }}/{{ .Version }}/
    username: goreleaser
```

Prerequisites:

- An HTTP server accepting HTTP requests
- A user + password with grants to upload an artifact using HTTP requests (if the server requires it)

### Target

The `target` is the template of the URL to upload the artifacts to (_without_ the name of the artifact).

An example configuration for `goreleaser` in upload mode `binary` with the target can look like

```yaml
- mode: binary
  target: 'http://some.server/some/path/example-repo-local/{{ .ProjectName }}/{{ .Version }}/{{ .Os }}/{{ .Arch }}{{ if .Arm }}{{ .Arm }}{{ end }}'
```

and will result in an HTTP PUT request sent to `http://some.server/some/path/example-repo-local/goreleaser/1.0.0/Darwin/x86_64/goreleaser`.

Supported variables:

- Version
- Tag
- ProjectName
- ArtifactName
- Os
- Arch
- Arm

> **Warning**: Variables `Os`, `Arch` and `Arm` are only supported in upload mode `binary`.

For `archive` mode, it will also included the `LinuxPackage` type which is
generated by `nfpm` and the like.

### Username

Your configured username needs to be valid against your HTTP server.

You can have the username set in the configuration file as shown above
or you can have it read from and environment variable.
The configured name of your HTTP server will be used to build the environment
variable name.
This way we support auth for multiple instances.
This also means that the `name` per configured instance needs to be unique
per GoReleaser configuration.

The name of the environment variable will be `UPLOAD_NAME_USERNAME`.
If your instance is named `production`, you can store the username in the
environment variable `UPLOAD_PRODUCTION_USERNAME`.
The name will be transformed to uppercase.

If a configured username is found in the configuration file, then the
environment variable is not used at all.

### Password

The password will be stored in a environment variable.
The configured name of your HTTP server will be used.
This way we support auth for multiple instances.
This also means that the `name` per configured instance needs to be unique
per GoReleaser configuration.

The name of the environment variable will be `UPLOAD_NAME_SECRET`.
If your instance is named `production`, you need to store the secret in the
environment variable `UPLOAD_PRODUCTION_SECRET`.
The name will be transformed to uppercase.

### Server authentication

You can authenticate your TLS server adding a trusted X.509 certificate chain
in your upload configuration.

The trusted certificate chain will be used to validate the server certificates.

You can set the trusted certificate chain using the `trusted_certificates`
setting the upload section with PEM encoded certificates on a YAML literal block
like this:

```yaml
uploads:
  - name: "some HTTP/TLS server"
    #...(other settings)...
    trusted_certificates: |
      -----BEGIN CERTIFICATE-----
      MIIDrjCCApagAwIBAgIIShr2zchZo+8wDQYJKoZIhvcNAQENBQAwNTEXMBUGA1UE
      ...(edited content)...
      TyzMJasj5BPZrmKjJb6O/tOtEIJ66xPSBTxPShkEYHnB7A==
      -----END CERTIFICATE-----
      -----BEGIN CERTIFICATE-----
      MIIDrjCCApagAwIBAgIIShr2zchZo+8wDQYJKoZIhvcNAQENBQAwNTEXMBUGA1UE
      ...(edited content)...
      TyzMJasj5BPZrmKjJb6O/tOtEIJ66xPSBTxPShkEYHnB7A==
      -----END CERTIFICATE-----
```

## Customization

Of course, you can customize a lot of things:

```yaml
# .goreleaser.yml
uploads:
  # You can have multiple upload instances.
  -
    # Unique name of your upload instance. Used to identify the instance.
    name: production

    # HTTP method to use.
    # Default: PUT
    method: POST

    # IDs of the artifacts you want to upload.
    ids:
    - foo
    - bar

    # Upload mode. Valid options are `binary` and `archive`.
    # If mode is `archive`, variables _Os_, _Arch_ and _Arm_ for target name are not supported.
    # In that case these variables are empty.
    # Default is `archive`.
    mode: archive

    # Template of the URL to be used as target of the HTTP request
    target: https://some.server/some/path/example-repo-local/{{ .ProjectName }}/{{ .Version }}/

    # Custom artifact name (defaults to false)
    # If enable, you must supply the name of the Artifact as part of the Target
    # URL as it will not be automatically append to the end of the URL, its
    # pre-computed name is available as _ArtifactName_ for example
    # target: https://some.server/some/path/example-repo-local/{{ .ArtifactName }};deb.distribution=xenial
    custom_artifact_name: true

    # User that will be used for the deployment
    username: deployuser

    # An optional header you can use to tell GoReleaser to pass the artifact's
    # SHA256 checksum within the upload request.
    # Default is empty.
    checksum_header: -X-SHA256-Sum

    # Upload checksums (defaults to false)
    checksum: true

    # Upload signatures (defaults to false)
    signature: true

   # Certificate chain used to validate server certificates
    trusted_certificates: |
      -----BEGIN CERTIFICATE-----
      MIIDrjCCApagAwIBAgIIShr2zchZo+8wDQYJKoZIhvcNAQENBQAwNTEXMBUGA1UE
      ...(edited content)...
      TyzMJasj5BPZrmKjJb6O/tOtEIJ66xPSBTxPShkEYHnB7A==
      -----END CERTIFICATE-----
```

These settings should allow you to push your artifacts into multiple HTTP servers.

> Learn more about the [name template engine](/templates).