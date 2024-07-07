
# AzureSignTool Action

## Description
Azure Sign Tool is similar to `signtool` in the Windows SDK, with the major difference being that it uses Azure Key Vault for performing the signing process. The usage is similar to `signtool`, except with a limited set of options for signing and options for authenticating to Azure Key Vault. <br />

The action uses the pre-compiled AzureSignTool v5.0 binary: https://github.com/vcsjones/AzureSignTool/tree/v5.0.0

<br /><br /><br />


## Example usage

```yaml
name: Build and Sign

on:
  push:
    branches: [main]

jobs:
  build-and-sign:
    runs-on: windows-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Build application
        run: ...

      - name: Sign files with AzureSignTool
        uses: CrzGames/azure-signtool-action@v1.0.1
        with:
          azure-key-vault-url: 'https://your-key-vault.vault.azure.net'
          azure-key-vault-client-id: ${{ secrets.AZURE_KEY_VAULT_APP_CLIENT_ID }}
          azure-key-vault-client-secret: ${{ secrets.AZURE_KEY_VAULT_APP_CLIENT_SECRET }}
          azure-key-vault-tenant-id: ${{ secrets.AZURE_KEY_VAULT_TENANT_ID }}
          azure-key-vault-certificate: 'your-certificate-name'
          timestamp-rfc3161: 'http://timestamp.digicert.com'
          timestamp-digest: 'sha512'
          file-digest: 'sha512'
          input-file-list: 'path/to/filelist.txt'
          files: 'path/to/your/files/*.exe path/to/another/file.dll path/to/your/directory'
          recursive: true
```

<br /><br /><br />


## Parameters

### Required Parameters 

- `azure-key-vault-url [required: yes]`: A fully qualified URL of the key vault with the certificate that will be used for signing. An example value might be `https://my-vault.vault.azure.net`.

- `azure-key-vault-certificate [required: yes]`: The name of the certificate used to perform the signing operation.

<br />

### Possibly Parameters 

- `input-file-list [required: possibly]`: Specifies a path to a text file which contains a list of files to sign, with one file per line in the text file. If this parameter is specified, it is combined with files directly specified on the command line. The distinct result of the two options is signed.

- `files [required: possibly]`: The paths to individual files, directories, or file patterns to be signed. Multiple paths can be specified, separated by spaces. This can include specific files, directories, or patterns such as *.exe.

- `recursive [required: possibly]`: Recursively sign files in directories. If set to true, the action will search for files to sign in all subdirectories of any directories specified in the `files` parameter.

- `azure-key-vault-client-id [required: possibly]`: This is the client ID used to authenticate to Azure, which will be used to generate an access token. This parameter is not required if an access token is supplied directly with the `azure-key-vault-accesstoken` option. If this parameter is supplied, `azure-key-vault-client-secret` and `azure-key-vault-tenant-id` must be supplied as well.

- `azure-key-vault-client-secret [required: possibly]`: This is the client secret used to authenticate to Azure, which will be used to generate an access token. This parameter is not required if an access token is supplied directly with the `azure-key-vault-accesstoken` option or when using managed identities with `azure-key-vault-managed-identity`. If this parameter is supplied, `azure-key-vault-client-id` and `azure-key-vault-tenant-id` must be supplied as well.

- `azure-key-vault-tenant-id [required: possibly]`: This is the tenant ID used to authenticate to Azure, which will be used to generate an access token. This parameter is not required if an access token is supplied directly with the `azure-key-vault-accesstoken` option or when using managed identities with `azure-key-vault-managed-identity`. If this parameter is supplied, `azure-key-vault-client-id` and `azure-key-vault-client-secret` must be supplied as well.

<br />

### Optional Parameters

- `azure-key-vault-accesstoken [required: possibly]`: An access token used to authenticate to Azure. This can be used instead of the `azure-key-vault-managed-identity`, `azure-key-vault-client-id` and `azure-key-vault-client-secret` options. This is useful if AzureSignTool is being used as part of another program that is already authenticated and has an access token to Azure.

- `azure-key-vault-managed-identity [required: possibly]`: Use the ambient Managed Identity to authenticate to Azure. This can be used instead of the `azure-key-vault-accesstoken`, `azure-key-vault-client-id` and `azure-key-vault-client-secret` options. This is useful if AzureSignTool is being used on a VM/service/CLI that is configured for managed identities to Azure. Important to mention is that this option leverages the DefaultAzureCredential class which is trying to get a token via multiple options including Visual Studio Credentials and Interactive Browser Authentication.

- `description [required: no]`: A description of the signed content. This parameter serves the same purpose as the /d option in the Windows SDK signtool. If this parameter is not supplied, the signature will not contain a description.

- `description-url [required: no]`: A URL with more information of the signed content. This parameter serves the same purpose as the /du option in the Windows SDK signtool. If this parameter is not supplied, the signature will not contain a URL description.

- `timestamp-rfc3161 [required: no]`: A URL to an RFC3161 compliant timestamping service. This parameter serves the same purpose as the /tr option in the Windows SDK signtool. This parameter should be used in favor of the --timestamp option. Using this parameter will allow using modern, RFC3161 timestamps which also support timestamp digest algorithms other than SHA1.

- `timestamp-authenticode [required: no]`: A URL to a legacy "Authenticode" timestamping service. This parameter serves the same purpose as the /t option in the Windows SDK signtool. Using a "Authenicode" timestamping service is deprecated. Instead, use the `timestamp-rfc3161` option.

- `timestamp-digest [required: no]`: The name of the digest algorithm used for timestamping. This parameter is ignored unless the `timestamp-rfc3161` parameter is also supplied. The default value is `sha256`. Possible values:
  - `sha1`
  - `sha256`
  - `sha384`
  - `sha512`

- `file-digest [required: no]`: The name of the digest algorithm used for hashing the file being signed. The default value is `sha256`. Possible values:
  - `sha1`
  - `sha256`
  - `sha384`
  - `sha512`

- `additional-certificates [required: no]`: A list of paths to additional certificates to aid in building a full chain for the signing certificate. AzureSignTool will build a chain, either as deep as it can or to a trusted root. This will also use the Windows certificate store, in addition to any certificates specified with this option. Specifying this option does not guarantee the inclusion of the certificate, only if it is part of the chain. To include multiple certificates, specify this option multiple times, such as `-ac file1.cer -ac file2.cer`. The files specified must be public certificates only. They cannot be PFX, PKCS12, or PFX files.

- `verbose [required: no]`: Include additional output in the log. This parameter does not accept a value and cannot be combined with the `quiet` option.

- `quiet [required: no]`: Do not print output to the log. This parameter does not accept a value and cannot be combined with the `verbose` option. The exit code of the process can be used to determine the success or failure of the sign operation.

- `continue-on-error [required: no]`: If multiple files to sign are specified, this flag will cause the signing process to move on to the next file when signing fails. This flag modifies the exit code of the program.

- `skip-signed [required: no]`: If a file is already signed, it will be skipped, rather than replacing the existing signature.

<br /><br /><br />


## Supported Formats

This tool uses the same mechanisms for signing as the Windows SDK signtool. It will support the same formats as signtool supports. However, the formats that azuresigntool and signtool support vary by operating system and which Subject Interface Packages are present on the system.

<br />

## Exit Codes

The exit code is an HRESULT. Successfully signing produces a result of S_OK ("0"). If all files fail to sign, the exit code is 0xA0000002. If some were signed successfully, the exit code is 0x20000001.

<br />

## Requirements

Windows >= 10 or Windows Server >= 2016 is required. <br />
Runners github-actions compatible : windows-2019, windows-2022 or windows-latest.

<br />

## Current Limitations

Dual signing is not supported. This appears to be a limitation of the API used.
