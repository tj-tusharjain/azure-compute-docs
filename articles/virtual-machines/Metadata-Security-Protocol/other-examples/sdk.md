# Private .NET SDK

Consume from MSFT dev feed:

1. add the dev feed to your .csproj

    ```xml
     <RestoreAdditionalProjectSources>
          https://pkgs.dev.azure.com/azure-sdk/public/_packaging/azure-sdk-for-net/nuget/v3/index.json
     </RestoreAdditionalProjectSources>
    ```

    Or add to your NuGet.Config as specified in the [dev feed doc](https://github.com/Azure/azure-sdk-for-net/blob/f2864e777b8f0067c3948909d8d18bba9c84bf64/CONTRIBUTING.md#nuget-package-dev-feed).

1. reference the nuget package in your project file

    ```xml
    <Project Sdk="Microsoft.NET.Sdk">
      <ItemGroup>
        <PackageReference Include="Azure.ResourceManager.Compute" Version="1.7.0-alpha.20240925.1" />
      </ItemGroup>
    </Project>
    ```

1. Consume the SDK. For example:

    ```c#
    using Azure.ResourceManager.Compute.Models;
    vmData.SecurityProfile = new SecurityProfile()
        {
            ProxyAgentSettings = new ProxyAgentSettings()
            {
                Enabled = true,
                Imds = new HostEndpointSettings()
                {
                    InVmAccessControlProfileReferenceId= "/subscriptions/6e6d1158-04fa-4954-89fc-92805ba952d6/resourcegroups/CPlatProxyAgentTest/Microsoft.Compute/galleries/PROXYAGENTGALLERY/InVMAccessControlProfiles/Windows_WS_Audit/versions/1.0.0",
                },
                WireServer = new HostEndpointSettings()
                {
                    InVmAccessControlProfileReferenceId = "/subscriptions/6e6d1158-04fa-4954-89fc-92805ba952d6/resourcegroups/CPlatProxyAgentTest/Microsoft.Compute/galleries/PROXYAGENTGALLERY/InVMAccessControlProfiles/Windows_Imds_Audit/versions/1.0.0",
                },
            }
        };
    ```

## Additional Languages

Please contact [Minnie Lahoti](mailto:minnielahoti@microsoft.com) and [Huimin Yan](mailto:Huimin.Yan@microsoft.com) to request SDK support in other programming languages.
