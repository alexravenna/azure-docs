---
title: Streamline Your Workflow with Dev Box customizations
description: Understand how to use Dev Box customizations to create ready-to-code configurations for your development teams and individual developers.
author: RoseHJM
ms.author: rosemalcolm
ms.service: dev-box
ms.custom:
  - ignite-2024
ms.topic: concept-article
ms.date: 05/09/2025

#customer intent: As a Dev Center Admin or Project Admin, I want to understand how to use Dev Box customizations so that I can create efficient, ready-to-code configurations for my development teams.
---

# Microsoft Dev Box customizations

[!INCLUDE [note-build-2025](includes/note-build-2025.md)]


Starting developers on a new project or team is often complex and time consuming. The Microsoft Dev Box customizations feature helps you streamline the setup of the developer environment. With customizations, you can configure ready-to-code workstations with the necessary applications, tools, repositories, code libraries, packages, and build scripts.

Dev Box customizations let you:
- Install the necessary tools and applications.
- Enforce organizational security policies.
- Ensure consistency across dev boxes.

Microsoft Dev Box offers two ways to use customizations. Team customizations are used to create a shared configuration for a team of developers. User customizations are used to create a personal configuration for an individual developer. The following table summarizes the differences between the two types of customizations.

| Feature                     | Team customizations       | User customizations |
|-----------------------------|---------------------------|---------------------------|
| **Configure on**            | Dev box pool             | Dev box                   |
| **Customizations apply to** | All dev boxes in pool    | Individual dev box        |
| **Easily shareable**        | Yes                      | No                        |
| **Customizations file name**| imagedefinition.yaml     | myfilename.yaml           |
| **Sourced from**            | Catalog or personal repository | Uploaded or from personal repository |
| **Supports key vault secrets** | Yes                  | Yes                       |

## What is a customization file?

Dev Box customizations use a YAML-formatted file to specify a list of tasks to apply from the dev center or a catalog when developers are creating a dev box. These tasks identify the catalog task and provide parameters like the name of the software to install. Developers create their own customization files or use shared ones. 
You can use secrets from your Azure key vault in your customization file to clone private repositories, or with any custom task you author that requires an access token.

## What are tasks?

Dev Box customization tasks are wrappers for PowerShell scripts. You use them to define reusable components that your teams can use in their customizations. 
WinGet and PowerShell tasks are available through the platform, and new ones can be added through a catalog.
Tasks run in either a system context or a user context after sign-in.
Team customizations are defined by project admins and can use both custom and built-in tasks.
User customizations can only use system tasks if the user is an administrator, or if the tasks are custom tasks preapproved by through a catalog.
When you're creating tasks, determine which of them need to run in a system context and which of them can run in a user context (after sign-in). 
When creating tasks, determine which need to run in a system context and which run in a user context after sign-in.

## Differences between team customizations and user customizations

Individual developers can upload a customization file when creating their dev box to control the development environment. Developers should use user customizations only for personal settings and apps. Tasks specified in the user customization file run only in the user context, after sign-in.
Developers use user customizations only for personal settings and apps.
Sharing common YAML files among developer teams is inefficient, leads to errors, and violates compliance policies. Dev Box team customizations allow developer team leads and IT admins to preconfigure customization files, eliminating the need for developers to find and upload these files when creating a dev box.

## How do customizations work?
Team customization and user customizations are both YAML-based files that specify a list of tasks to apply when creating a dev box. Select the appropriate tab to learn more about how each type of customization works.

# [Team customizations](#tab/team-customizations)
### How do team customizations work?

You can use team customizations to define a shared Dev Box configuration for each of your development teams without having to invest in setting up an imaging solution like Packer or Azure virtual machine (VM) image templates. Team customizations provide a lightweight alternative that allows central platform engineering teams to delegate Dev Box configuration management to the teams that use them.

Team customizations also offer an in-built way of optimizing your team's Dev Box customizations by flattening them into a custom image. You use the same customization file, without the need to manage added infrastructure or maintain image templates.

When you configure Dev Box team customizations for your organization, careful planning and informed decision-making are essential. The following diagram provides an overview of the process and highlights key decision points.


:::image type="content" source="media/concept-what-are-team-customizations/dev-box-customizations-workflow.svg" alt-text="Diagram that shows the workflow for Dev Box team customizations, including steps for planning, configuring, and deploying customizations." lightbox="media/concept-what-are-team-customizations/dev-box-customizations-workflow.svg":::

#### Configure your Dev Box service for team customizations

To set up your Dev Box service to support team customizations, follow these steps:

1. **Configure your dev center**:
   1. Enable project-level catalogs.
   1. Assign permissions for project admins.

1. **Decide whether to use a catalog with custom reusable components**:
   1. **Built-in**:
      1. Use PowerShell or WinGet statements.
   1. **Catalog**:
      1. Host in Azure Repos or GitHub.
      1. Add tasks.
      1. Attach to a dev center.

1. **Create a customization file**:
   1. Create a YAML file named `imagedefinition.yaml`.

1. **Specify an image in a dev box pool**:
   1. Create or modify a dev box pool and specify `imagedefinition.yaml` as the image definition.

1. **Choose how you'll use the image definition**:
   1. Build the image each time you create a dev box.
   1. Optimize the image for team customizations.

1. **Create a dev box**:
   1. Use the developer portal to create your dev box from the configured pool.

For more information, see [Write an image definition file for Dev Box team customizations](how-to-write-image-definition-file.md).

# [User customizations](#tab/user-customizations)
### How do user customizations work?
Individual developers can attach a YAML-based customization file when creating their dev box to control the development environment. Developers use user customizations only for personal settings and apps.  

:::image type="content" source="media/concept-what-are-team-customizations/dev-box-user-customizations-workflow.svg" alt-text="Diagram that shows the workflow for Dev Box user customizations, including steps for planning, configuring, and deploying customizations." lightbox="media/concept-what-are-team-customizations/dev-box-user-customizations-workflow.svg":::

#### Configure your Dev Box service for user customizations

To set up your Dev Box service to support user customizations, follow these steps:

1. **Use a PowerShell and WinGet tasks**:
   1. Platform supports PowerShell and WinGet.
   1. No catalog required.
   1. No further configuration required.

1. **Create a customization file**:
   1. Create a YAML-based customization file.

1. **Create a dev box**:
   1. Use the developer portal to create your dev box from the configured pool.
   1. Upload and validate your customization file during the dev box creation process.

1. **Dev box creation**:
   1. The dev box is created with the specified customizations.

For more information, see [Write a user customization file for a dev box](how-to-write-user-customization-file.md).

---

## Related content

- [Quickstart: Create a dev box by using team customizations](quickstart-team-customizations.md)
- [Configure imaging for Dev Box team customizations](how-to-configure-customization-imaging.md)

