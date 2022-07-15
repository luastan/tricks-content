---
title: Data minning
description: Recon
position: 6
---

# Data mining

If an user has got access to any storage account it is useful to use [Storage Explorer](https://github.com/microsoft/AzureStorageExplorer) because it might have more permissions to access information.

## Connecting with a SASToken

A SAS Token (Shared Access Signature) is a way to granularly control how a client can access Azrue data. You can control many things such as what resources the client can access, what permission the client has, how long the token is valid for and more.

It is usually used to secure Azure Storage Accounts.

It usually has a similar structure to this:

`?sv=2019-07-07&ss=bfqt&srt=sco&sp=rl&se=2022-05-18T20:57:18Z&spr=https&sig=%2FauZD65pCN7UkZMTBG2pyePIJTssseTGiEl%2BOEazU4M%3D`

This token can be used with Azure Storage Explorer adding the full URL

```
https://secretblpstorage.blob.core.windows.net/blueprint?sv=2019-07-07&ss=bfqt&srt=sco&sp=rl&se=2022-05-18T20:57:18Z&spr=https&sig=%2FauZD65pCN7UkZMTBG2pyePIJTssseTGiEl%2BOEazU4M%3D
```

Or using command line

```powershell
$env:AZCOPY_CRED_TYPE = "Anonymous";
./azcopy.exe copy "https://secretblpstorage.blob.core.windows.net/blueprint/reader.txt?sv=2019-07-07&ss=bfqt&srt=sco&sp=rl&se=2022-05-18T20%3A57%3A18Z&spr=https&sig=%2FauZD65pCN7UkZMTBG2pyePIJTssseTGiEl%2BOEazU4M%3D" "C:\Users\STUDEN~1\AppData\Local\Temp\2\16503169292451\reader.txt" --overwrite=prompt --check-md5 FailIfDifferent --from-to=BlobLocal --blob-type BlockBlob --recursive;
$env:AZCOPY_CRED_TYPE = "";
```