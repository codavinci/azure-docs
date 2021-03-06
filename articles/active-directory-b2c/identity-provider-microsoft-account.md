---
title: Set up sign-up and sign-in with a Microsoft Account
titleSuffix: Azure AD B2C
description: Provide sign-up and sign-in to customers with Microsoft Accounts in your applications using Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg

ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 01/27/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
---

# Set up sign-up and sign-in with a Microsoft account using Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

## Prerequisites

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## Create a Microsoft account application

To enable sign-in for users with a Microsoft account in Azure Active Directory B2C (Azure AD B2C), you need to create an application in [Azure portal](https://portal.azure.com). For more information, see [Register an application with the Microsoft identity platform](../active-directory/develop/quickstart-register-app.md). If you don't already have a Microsoft account, you can get one at [https://www.live.com/](https://www.live.com/).

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Make sure you're using the directory that contains your Azure AD tenant by selecting the **Directory + subscription** filter in the top menu and choosing the directory that contains your Azure AD tenant.
1. Choose **All services** in the top-left corner of the Azure portal, and then search for and select **App registrations**.
1. Select **New registration**.
1. Enter a **Name** for your application. For example, *MSAapp1*.
1. Under **Supported account types**, select **Accounts in any organizational directory (Any Azure AD directory - Multitenant) and personal Microsoft accounts (e.g. Skype, Xbox)**.

   For more information on the different account type selections, see [Quickstart: Register an application with the Microsoft identity platform](../active-directory/develop/quickstart-register-app.md).
1. Under **Redirect URI (optional)**, select **Web** and enter `https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/oauth2/authresp` in the text box. Replace `<tenant-name>` with your Azure AD B2C tenant name.
1. Select **Register**
1. Record the **Application (client) ID** shown on the application Overview page. You need the client ID when you configure the identity provider in the next section.
1. Select **Certificates & secrets**
1. Click **New client secret**
1. Enter a **Description** for the secret, for example *Application password 1*, and then click **Add**.
1. Record the application password shown in the **Value** column. You need the client secret when you configure the identity provider in the next section.

::: zone pivot="b2c-user-flow"

## Configure Microsoft as an identity provider

1. Sign in to the [Azure portal](https://portal.azure.com/) as the global administrator of your Azure AD B2C tenant.
1. Make sure you're using the directory that contains your Azure AD B2C tenant by selecting the **Directory + subscription** filter in the top menu and choosing the directory that contains your tenant.
1. Choose **All services** in the top-left corner of the Azure portal, search for and select **Azure AD B2C**.
1. Select **Identity providers**, then select **Microsoft Account**.
1. Enter a **Name**. For example, *MSA*.
1. For the **Client ID**, enter the Application (client) ID of the Azure AD application that you created earlier.
1. For the **Client secret**, enter the client secret that you recorded.
1. Select **Save**.

## Add Microsoft identity provider to a user flow 

1. In your Azure AD B2C tenant, select **User flows**.
1. Click the user flow that you want to add the Microsoft identity provider.
1. Under the **Social identity providers**, select **Microsoft Account**.
1. Select **Save**.
1. To test your policy, select **Run user flow**.
1. For **Application**, select the web application named *testapp1* that you previously registered. The **Reply URL** should show `https://jwt.ms`.
1. Click **Run user flow**

::: zone-end

::: zone pivot="b2c-custom-policy"

## Configuring optional claims

If you want to get the `family_name` and `given_name` claims from Azure AD, you can configure optional claims for your application in the Azure portal UI or application manifest. For more information, see [How to provide optional claims to your Azure AD app](../active-directory/develop/active-directory-optional-claims.md).

1. Sign in to the [Azure portal](https://portal.azure.com). Search for and select **Azure Active Directory**.
1. From the **Manage** section, select **App registrations**.
1. Select the application you want to configure optional claims for in the list.
1. From the **Manage** section, select **Token configuration (preview)**.
1. Select **Add optional claim**.
1. Select the token type you want to configure.
1. Select the optional claims to add.
1. Click **Add**.

## Create a policy key

Now that you've created the application in your Azure AD tenant, you need to store that application's client secret in your Azure AD B2C tenant.

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Make sure you're using the directory that contains your Azure AD B2C tenant. Select the **Directory + subscription** filter in the top menu and choose the directory that contains your tenant.
1. Choose **All services** in the top-left corner of the Azure portal, and then search for and select **Azure AD B2C**.
1. On the Overview page, select **Identity Experience Framework**.
1. Select **Policy Keys** and then select **Add**.
1. For **Options**, choose `Manual`.
1. Enter a **Name** for the policy key. For example, `MSASecret`. The prefix `B2C_1A_` is added automatically to the name of your key.
1. In **Secret**, enter the client secret that you recorded in the previous section.
1. For **Key usage**, select `Signature`.
1. Click **Create**.

## Configure Microsoft as an identity provider

To enable users to sign in using a Microsoft account, you need to define the account as a claims provider that Azure AD B2C can communicate with through an endpoint. The endpoint provides a set of claims that are used by Azure AD B2C to verify that a specific user has authenticated.

You can define Azure AD as a claims provider by adding the **ClaimsProvider** element in the extension file of your policy.

1. Open the *TrustFrameworkExtensions.xml* policy file.
1. Find the **ClaimsProviders** element. If it does not exist, add it under the root element.
1. Add a new **ClaimsProvider** as follows:

    ```xml
    <ClaimsProvider>
      <Domain>live.com</Domain>
      <DisplayName>Microsoft Account</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="MSA-MicrosoftAccount-OpenIdConnect">
          <DisplayName>Microsoft Account</DisplayName>
          <Protocol Name="OpenIdConnect" />
          <Metadata>
            <Item Key="ProviderName">https://login.live.com</Item>
            <Item Key="METADATA">https://login.live.com/.well-known/openid-configuration</Item>
            <Item Key="response_types">code</Item>
            <Item Key="response_mode">form_post</Item>
            <Item Key="scope">openid profile email</Item>
            <Item Key="HttpBinding">POST</Item>
            <Item Key="UsePolicyInRedirectUri">false</Item>
            <Item Key="client_id">Your Microsoft application client ID</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_MSASecret" />
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="oid" />
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
            <OutputClaim ClaimTypeReferenceId="surName" PartnerClaimType="family_name" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" PartnerClaimType="iss" />
            <OutputClaim ClaimTypeReferenceId="email" />
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName" />
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName" />
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId" />
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId" />
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin" />
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

1. Replace the value of **client_id** with the Azure AD application's *Application (client) ID* that you recorded earlier.
1. Save the file.

You've now configured your policy so that Azure AD B2C knows how to communicate with your Microsoft account application in Azure AD.

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="MicrosoftAccountExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="MicrosoftAccountExchange" TechnicalProfileReferenceId="MSA-MicrosoftAccount-OpenIdConnect" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

[!INCLUDE [active-directory-b2c-test-relying-party-policy](../../includes/active-directory-b2c-test-relying-party-policy-user-journey.md)]

::: zone-end
