---
title: VCPKG Binary Caching with Nuget
date: 2022-12-22T19:18:32.295Z
draft: false
longform: true
tags:
  - vcpkg
---


When using vcpkg on projects with many or large dependencies, a large portion of your total build time can be just from building these dependencies even though the resulting artifacts don't change between compiles.
On a local development environment this is usually an upfront time cost as vcpkg will by default save the artifacts to a local per-user cache but for CI environments you don't necessarily have the persistant storage of this cache between runs.

To address this there is support for using alternative caching solutions. Using these caching solutions also lets developers more easily work from multiple machines since there is no longer a per-machine cost for compiling dependencies.

The official documentation for binary caching can be found here: https://learn.microsoft.com/en-us/vcpkg/users/binarycaching
For this example we will use a self hosted NuGet repository hosted using the new package registry support on Gitea. The same steps apply for any other platform offering NuGet hosting.

The first step is to add the registry to NuGet. On Linux systems this requires mono to be installed.
What we need is the URL for the NuGet feed and credentials to access the system. 
For a CI system the hardcoded application token can be replaced by whatever means you use for secrets management.
We use 'vcpkg fetch' to handle installing a repository-local copy of nuget with a version supported by vcpkg.
If you have an appropriate version of nuget already installed then this can be replaced with the normal nuget command.



P﻿owershell:
```powershell
SOURCE=Gitea
USERNAME=john_doe
PASS=pam_token
SOURCE_URL=https://<gitea_url>/api/packages/$USERNAME/nuget/index.json
& $(vcpkg fetch nuget) add source --name $SOURCE --username $USERNAME --password $PASS $SOURCE_URL
```

B﻿ash:


```bash
SOURCE=Gitea
USERNAME=john_doe
PASS=pam_token
SOURCE_URL=https://<gitea_url>/api/packages/$USERNAME/nuget/index.json
mono $(vcpkg fetch nuget | tail -n 1) add source --name $NAME --username $USERNAME --password $PASS $SOURCE_URL
```

Now that the repository has been added to NuGet we can set the environment variable VCPKG_BINARY_SOURCES to use this cache.

P﻿owershell:


```powershell
[Environment]::SetEnvironmentVariable("VCPKG_BINARY_SOURCES", "nuget,$SOURCE,readwrite", [EnvironmentVariableTarget::User])
````

B﻿ash:


```bash
echo "export VCPKG_BINARY_SOURCES=nuget,$SOURCE,readwrite" >> ~/.bashrc
```

If you want to completely disable the local caching, you can prepend 'clear;' to the start of the configuration string.

Now if you install some package, it will first check the registry to see if the binaries are already available, and if they aren't it will download and compile it on your local machine and then upload the binaries for future use.

Additional Notes:
If you are self-hosting a NuGet repository behind a proxy, you might need to adjust your proxy's max body size to allow for uploading large artifacts.
For nginx this can be set with the parameter client_max_body_size.