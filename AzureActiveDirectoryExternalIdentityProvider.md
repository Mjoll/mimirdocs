# Configure Azure Active Directory as an external identity provider for Mimir

## Overview of registration process

1. Obtain the following information from Mjoll support:
    * Identifier (Entity ID)
    * Reply URL (Assertion Consumer Service URL)
2. Configure Mimir as an Azure Active Directory Enterprise Application
3. Provide the following information to Mjoll support:
    * The “Federated Metadata XML” file.
    * The email domain address you claim ownership of.
4. Wait for Mjoll support to:
    1. Verify that you own the email domain address given.
    2. Register your Azure Active Directory as an external identity provider 

## Obtain information from Mjoll support

Before you start configuring Azure Active Directory as an external identity provider for Mimir, you will need to obtain the following Security Assertion Markup Language (SAML) related information from Mjoll support:

* Identifier (Entity ID)
* Reply URL (Assertion Consumer Service URL)

This information will vary depending on which region of the world you reside in.

## Configure Mimir as an Azure Active Directory Enterprise Application

### Create Enterprise Application for Mimir

Go to Microsoft Azure → Azure Active Directory → Manager: Enterprise Applications → New application → Create your own application

1. For the “What’s the name of your app?” prompt, we suggest typing in “Mimir”, but the choice is yours. We will use “_{Mimir}_” when referring to the name you chose in the instructions below.
2. For the “What are you looking to do with your application?”, choose the option “Integrate any other application you don’t find in the gallery (Non-gallery)”.
3. Click the “Create” button.

### Set up Basic SAML Configuration

Go to Microsoft Azure → Azure Active Directory → Enterprise application → _{Mimir}_ → Manage: Single sign-on → SAML → Set up Single Sign-On with SAML → Basic SAML Configuration → Edit

1. For the “Identifier (Entity ID)” and the “Reply URL (Assertion Consumer Service URL” prompt, enter in the information you obtained from Mjoll support.
2. Click “Save”.

### Set up user attributes and claims

Go to Microsoft Azure → Azure Active Directory → Enterprise application → _{Mimir}_ → Manage: Single sign-on → SAML → Set up Single Sign-On with SAML → User Attributes & Claims → Edit

Verify that the following claim required by SAML has been set up. This should be the default configuration, meaning that you don’t have to edit the claims.

| Claim name | Value / Source Attribute |
| ---------- | ------------------------ |
| Unique User Identifier (Name ID) | user.userprincipalname |

Verify that the following additional claims required by the Mimir integration have been set up.  This should be the default configuration, meaning that you don’t have to edit the claims.

| Claim name | Value / Source Attribute |
| --------- | ------------------------ |
| http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress | user.mail |
| http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name | user.userprincipalname |
| http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname | user.givenname |
| http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname | user.surname |

Optionally, to allow Azure Active Directory groups to be used to assign Mimir user permissions, add the following group claim. This claim is not part of the default configuration, meaning that if you’d like to make Azure Active Directory groups available for use in Mimir, you will need to add this claim.

| Claim name | Value / Source Attribute |
| --------- | ------------------------ |
| http://schemas.microsoft.com/ws/2008/06/identity/claims/groups | user.groups [All] |

1. Click “Add a group claim”.
2. For the “Which groups associated with the user should be returned in the claim” prompt, select “Groups assigned to the application”.
3. For the “Source attribute” prompt choose the default “Group ID” option.
4. Click “Save”.

### Assign users to the enterprise application

Before any users can use Mimir, Azure Active Directory requires you to assign users to the enterprise application representing Mimir in Active Directory.

Azure Active Directory provides several ways of managing access to applications, however this document only describes direct assignment of users.

Go to Microsoft Azure → Azure Active Directory → Enterprise application → _{Mimir}_ → Manage: Users and groups → Add user/group

1. Click the “Users: None Selected” link.
2. Click users to add.
3. Click the “Select” button.
4. Click the “Assign” button.

## Provide information to Mjoll support

Go to Microsoft Azure → Azure Active Directory → Enterprise application → _{Mimir}_ → Manage: Single sign-on → SAML → Set up Single Sign-On with SAML → SAML Signing Certificate → Federation Metadata XML

Click the “Download” link for “Federated Metadata XML”, and download and save the XML file from this link.

Decide upon the email domain that you want Mimir to recognize as belonging to your organization. As an example, if your users in their daily work use email addresses such as _johndoe@example.com_, then the email domain name _example.com_ might be appropriate.

Only email addresses that you can prove that you own are applicable. You cannot use email domain names shared with other organizations. You cannot for example claim _gmail.com_ as your domain name.

Send the following information to Mjoll support:

* The “Federated Metadata XML” file you downloaded.
* The email domain name you claim ownership of.

## Assigning Mimir Permissions to Azure Active Directory groups

Permission to perform Mimir action is assigned to an Azure Active Directory group by assigning permissions to matching group definitions in Mimir.

### Creating a matching Mimir group definition

Go to Azure → Azure Active Directory → Groups

Collect the group identity and display label of the desired group from the "Object Id" and "Name" properties of an Azure Active Directory group.

| Group information | Azure Active Directory group property | Mimir group property |
| ----------------- | ------------------------------------- | -------------------- |
| Identity          | Object Id                             | Name                 |
| Display label     | Name                                  | Short description    |

Go to Mimir → Profile menu → My organization → Groups → "+" (Add new group)

Create a matching group definition in Mimir by entering in its identity as the "Group name", and the display label as the "Short description".

Note that while both Azure Active Directory and Mimir has group properties called "Name", they serve different purposes, since one the Azure Active Directory group name represents a human readable display label, while the Mimir group name represents the identity of the group, used to match the Mimir group definition with the Azure Active Directory group definition.

Technically, only matching identifiers are needed to attach Azure Active Directory groups to permissions assigned to Mimir groups. However, we recommend also matching the display labels to make it easier for administrators to see what group definitions exists at a glance.

The effective permissions of a user signing into Mimir using an external Azure Active Directory account is the aggregate permissions of all the Mimir groups matching Azure Active Directory groups they are members of, and also the permissions from the Mimir group named "Organization", if such a group exists.

### Assigning email addresses to Azure Active Directory users

Mimir requires users to have email addresses.

Go to Azure → Azure Active Directory → Users → {User} → Edit → Contact Information → Email

Enter the email address which you wish Mimir to associate with the user in the "Email" field.

Click the "Save" button.

### Signing in using Azure administrator accounts

We've observed that it is possible to assign Azure administrator accounts to Enterprise Applications in Azure Active Directory. Signing into Mimir using these accounts seems to work, but we've observed that Mimir does not receive any group information from these accounts. If you encounter this problem, you have two options:

 1. Create a regular account within Azure Active Directory, and assign it to the Active Directory groups with matching Mimir groups as a way of assigning them Mimir permissions.
 2. Create and configure a Mimir "Organization" group which provides permissions. Any permissions assigned to this Mimir group will be given to all users authenticated by Azure Active Directory, including any Azure administrator accounts.

### Refreshing security token

Users may need to log out of both Mimir and their Azure Active Directory account and then log back in again for any of the following changes to take effect:
 * Editing the mapping of groups to SAML claims in Azure Active Directory.
 * Editing group membership in Azure Active Directory.
 * Creation of matching groups in Mimir.
 * Editing of Mimir permissions attached to a group in Mimir.

In addition, it can take up to five minutes for group permission changes to propagate and take effect everywhere.

Therefore, if you've made any of these changes, perform the following steps:
 * Sign out of the Mimir UI.
 * Sign out of your Azure Active Directory account. Follow the instructions specific to your organization, or sign out from https://myaccount.microsoft.com/
 * Wait until at least five minutes has passed since any administrative account or group change.
 * Sign in to the Mimir UI using your Azure Active Directory account.

## Troubleshooting

### User is not assigned to a role for the application

Problem: AADSTS50105: The signed in user '...' is not assigned to a role for the application '...'(...).

Cause: Azure Active Directory has not been instructed to give the signed in user access to the enterprise application representing Mimir.

Resolution: Follow the instructions in section “Assign users to the enterprise application”.

### The reply URL specified does not match the URLs configured

Problem: AADSTS50011: The reply URL specified in the request does not match the reply URLs configured for the application: '...'.

Cause: Either the Reply URL (Assertion Consumer Service URL) or Identifier (Entity ID) has been incorrectly configured.

Resolution #1: Follow the instructions in section “Set up Basic SAML Configuration”.

Resolution #2: Ask Mjoll support to verify the Identifier (Entity ID) and Reply URL (Assertion Consumer Service URL) they have given you match the geographical region of your tenant instance.

### Invalid user attributes: email: The attribute is required

Problem: server_error: Error in SAML response processing: Invalid user attributes: email: The attribute is required

Cause: Mimir requires users to have email addresses attached to them, however, the SAML response from Azure Active Directory did not contain any email address.

Resolution #1: Follow the instructions in the section "Assigning email addresses to Azure Active Directory users".

Resolution #2: Follow the instructions in the section "Set up user attributes and claims".

### Missing group permissions

Problem: After logging in, the user does not have permission to perform all the Mimir actions that they expect.

Cause #1: The security token they were issued during login does not contain up-to-date information.

Resolution #1: Follow the instructions in section "Refreshing Security Token".

Cause #2: The Azure Active Directory group Object Id does not match the Mimir group name.

Resolution #2: Follow the instructions in section "Creating a matching Mimir group definition".

Cause #3: Azure Active Directory does not expose group membership for Azure administrator accounts to Mimir.

Resolution #3: Follow the instructions in section "Signing in using Azure administrator accounts".
