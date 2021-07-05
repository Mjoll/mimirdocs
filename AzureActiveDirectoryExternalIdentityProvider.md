# Configure Active Directory as an external identity provider for Mimir

## Overview of registration process

1. Obtain the following information from Mjoll support:
    1. Identifier (Entity ID)
    2. Reply URL (Assertion Consumer Service URL)
2. Configure Mimir as an Azure Active Directory Enterprise Application
3. Provide the following information to Mjoll support:
    3. The “Federated Metadata XML” file.
    4. The email domain address you claim ownership of.
4. Wait for Mjoll support to:
    5. Verify that you own the email domain address given.
    6. Register your Azure Active Directory as an external identity provider for 

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

<table>
  <tr>
   <td><strong>Claim name</strong>
   </td>
   <td><strong>Value / Source Attribute</strong>
   </td>
  </tr>
  <tr>
   <td>Unique User Identifier (Name ID)
   </td>
   <td>user.userprincipalname
   </td>
  </tr>
</table>

Verify that the following additional claims required by the Mimir integration have been set up.  This should be the default configuration, meaning that you don’t have to edit the claims.

<table>
  <tr>
   <td><strong>Claim name</strong>
   </td>
   <td><strong>Value / Source Attribute</strong>
   </td>
  </tr>
  <tr>
   <td>http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress
   </td>
   <td>user.mail
   </td>
  </tr>
  <tr>
   <td>http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name
   </td>
   <td>user.userprincipalname
   </td>
  </tr>
  <tr>
   <td>http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
   </td>
   <td>user.givenname
   </td>
  </tr>
  <tr>
   <td>http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname
   </td>
   <td>user.surname
   </td>
  </tr>
</table>

Optionally, to allow Azure Active Directory groups to be used to assign Mimir user permissions, add the following group claim. This claim is not part of the default configuration, meaning that if you’d like to make Azure Active Directory groups available for use in Mimir, you will need to add this claim.

<table>
  <tr>
   <td><strong>Claim name</strong>
   </td>
   <td><strong>Value / Source Attribute</strong>
   </td>
  </tr>
  <tr>
   <td>http://schemas.microsoft.com/ws/2008/06/identity/claims/groups
   </td>
   <td>user.groups [All]
   </td>
  </tr>
</table>

1. Click “Add a group claim”.
2. For the “Which groups associated with the user should be returned in the claim” prompt, select “All groups”.
3. For the “Source attribute” prompt choose the default “Group ID” option.
4. Click “Save”.

### Assign users to the enterprise application

Before any users can use the Mimir, Azure Active Directory requires you to assign users to the enterprise application representing Mimir in Active Directory.

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