---
title: Renew Trusted Signing Identity Validation
description: How-to rerenew a Trusted Signing Identity Validation. 
author: mehasharma 
ms.author: mesharm 
ms.service: trusted-signing 
ms.topic: how-to 
ms.date: 04/12/2024 
---

# Renew Trusted Signing Identity Validation 

You can check the expiration date of your Identity Validation on the Identity Validation page. You can renew your Trusted Signing Identity Validation 60 days before the expiration. A notification is to the primary and secondary email addresses with the reminder to renew your Identity Validation.
**Identity Validation can only be completed in the Azure portal – it can not be completed with Azure CLI.**

>[!Note]
>Failure to renew Identity Validation before the expiration date will stop certificate renewal, effectively halting the signing process associated with those specific certificate profiles.

1. Navigate to your Trusted Signing account in the [Azure portal](https://portal.azure.com/).
2. Confirm you have the **Trusted Signing Identity Verifier role**.
    - To learn more about Role Based Access management (RBAC) access management, see [Assigning roles in Trusted Signing](tutorial-assign-roles.md).
3. From either the Trusted Signing account overview page or from Objects, select **Identity Validation**.
4. Select the Identity Validation request that needs to be renewed. Select **Renew** on the top. 

:::image type="content" source="media/trusted-signing-renew-identity-validation.png" alt-text="Screenshot of trusted-signing-renew-identity-validation.png." lightbox="media/trusted-signing-renew-identity-validation.png":::

5. If you encounter validation errors while renewing through the renew button or if Identity Validation is Expired, you need to create a new Identity Validation. 
    - To learn more about creating new Identity Validation, see [Quickstart](quickstart.md). 
6. After the Identity Validation status changes to Completed.
7. To ensure you can continue with your existing metadata.json.
    - Navigate back to the trusted signing account overview page or from Objects, select **Certificate Profile**.
    - On the **Certificate Profiles**, delete the existing cert profile associated to the Identity Validation expiring soon:
    - Create new cert profile with the same name.
    - Select the Identity Validation from the pull-down. Once the certificate profile is created successfully, signing resumes requiring no configuration changes on your end.