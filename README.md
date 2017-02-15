# yale-sharepoint-cas
An updated version of Yale's CAS login for Microsoft Sharepoint

Based on http://eduyalesomauth.codeplex.com/

(*Note: This file is still being converted to Markdown. The end is not finished.*)

## License

Copyright (c) 2008 Brendan Kennedy Turnbull
Copyright (c) 2008 Yale University School of Management

READ [LICENSE](LICENSE) file for full license.

## Project Description
 
The edu.yale.som.auth .NET solution:

- Combines different authentication and authorization providers to implement Hybrid Authentication from a single login page for SharePoint 2007 (MOSS) and ASP.NET web sites.
- Currently released authentication method implementations:
 - Yale University Central Authentication Service (CAS)
 - Forms Based Authentication (FBA)
 - Active Directory (AD)

- Currently released authorization method implementations:
 - Forms Based Authorization

- Integrates MOSS or standard ASP.NET web applications with a CAS authentication method implementation to allow users to authenticate via CAS and authorize against FBA Roles.
- Exposes a consistent login experience for users across all your MOSS and ASP.NET web applications
- Can display multiple authentication methods or can be configured to always default to a single method, e.g. CAS.
- Provides user processing for FBA Membership, Profile and Roles dynamically during login and/or through batch process via custom FBA administration web parts.
- Allows you to map your existing AD group memberships to FBA Roles for authorization against those Roles in MOSS or standard ASP.NET web applications 
- Simplifies MOSS site design, deployment and management by allowing you to expose multiple authentication and authorization methods in a single MOSS web site Zone.
- Deployed to SharePoint as a Feature that applies configuration changes upon Feature activation.
- Utilizes a plug in interface design that allows you to implement additional authentication and authorization classes to extend the login functionality.

 - The edu.yale.som.auth architecture, provides the interfaces, collections and logic that uncouples the traditional one to one relationship between a web application and its authentication/authorization method.
 - This uncoupling allows for multiple authentication/authorization methods to be utilized by a single web application.
 - The application consists of a login web page, along with its supporting web and application configuration files, and the class library edu.yale.som.auth that serves as the code behind implementation for that login page and Admin web parts. 
 - The authentication methods are selected by the end user from a single login page.
 - The selectable authentication methods are configured via an xml configuration file that is read (de-serialized) by the application when the login page is loaded. 
 - Each authentication and authorization provider is incorporated seperately as a class that implements the defined authentication or authorization interfaces.
 - The interfaces allow for the authentication and authorization classes (implementations) to be programatically "plugged in" or "unplugged" from the login web page by changing the xml configuration file AuthConfig.config.
 - Once an authentication provider is implemented in the library, it can be associated with one of the authentication implementation classes and then enabled in the configuration to appear as a login option to the end user.

## Requirements

- Microsoft Windows Server 2008 / IIS 7
- Microsoft Visual Studio 2008
- Microsoft SQL Server 2005 or later
- Microsoft Active Directory
- Central Authentication Service: http://www.ja-sig.org/products/cas/ 
- Microsoft SharePoint 2007
- WSPBuilder: http://www.codeplex.com/wspbuilder
- SharePoint 2007 Shared Services Provider User Profile Importer: http://www.codeplex.com/MOSSProfileImport

## Config

### IIS Config

#### Configure IIS security context on the Default zone

1. Sites\SharePoint - 80\Authentication
    1. Anonymous Authentication
        1. Click Enable
        1. Click Edit
            1. Ensure "Application Pool Identity" is selected.
    1. ASP.NET Impersonation
        1. Click Enable
        1. Click Edit
            1. Ensure "Authenticated User" is selected.
    1. Windows Authentication
        1. Click Advanced Settings
        1. Uncheck "Enable Kernal-mode authentication"
1. Set the application pool process identity of the SharePoint content Default zone site to the [SharePoint domain service account].

#### SharePoint Config

1. SharePoint Central Admin\Operations\Security Configuration\Service accounts 
    1. Select Web application pool
    1. Web service:  "Windows SharePoint Services Web Application"
    1. Application Pool: "SharePoint - 80" 
    1. Select Configurable
    1. User name: [SharePoint domain service account]
    1. Click OK
    1. Click ok to the Kerberos SPN warnings
1. Operations\Security Configuration\Update farm administrator's group
    1. New\Add Users
    1. Enter [SharePoint domain service account]
    1. Select "Add users to the SharePoint group" "Farm Administrators [Full Control]"
    1. Click OK

### Kerberos Config

1. Add a Service Principal Name (SPN) to the [SharePoint domain service account] for your HTTP service
1. The following steps allows kerberos authentication to be delegated by the [SharePoint domain service account]
1. Replace "MY_HOST_NAME" in the following example with the host name of your SharePoint site as in "http://MY_HOST_NAME")
1. Replace "DOMAIN\SERVICE_ACCOUNT" with the [SharePoint domain service account]. 
1. A system admin with rights to modify Active Directory in your domain will need to run the following commands.

        C:\Windows\system32>setspn -A HTTP/MY_HOST_NAME DOMAIN\SERVICE_ACCOUNT
        C:\Windows\system32>setspn -A HTTP/MY_HOST_NAME.YOUR_DOMAIN DOMAIN\SERVICE_ACCOUNT

### Local Access Config

1. Add [SharePoint domain service account] to the local Administrators group of the SharePoint WFE

### Database Config

#### Setup the ASP.NET Forms Based Authentication (FBA) Provider Data Store

1. Create an empty SQL Server database called "Core" 
1. At a command prompt running on the SQL Server, run the following command:

        C:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\aspnet_regsql -S (local) -E -A all -d Core

#### Setup the edu.yale.som.lib database

1. Create an empty SQL Server database called "edu.yale.som.lib" 
1. Run the database scripts in `obj\edu.yale.som.auth\trunk\database\edu.yale.som.lib` on the `edu.yale.som.lib` database

#### Setup the facebook database

NOTE: You can replace the facebook database with your own human resources (HR) database so long as the edu.yale.som.lib database view `dbo.View_edu_yale_som_auth_Users.View` can pull from your HR data source. 

1. Create an empty SQL Server database called "facebook"
1. Run the database scripts in `obj\edu.yale.som.auth\trunk\database\facebook` on the facebook database

#### Grant access to the databases

Grant the [SharePoint domain service account]:

1. Full access to all of the database roles in the the Core database that have names starting with "aspnet_"
1. datareader access to databases:
    - EduYaleSomLib 
    - Facebook 
1. Execute permission to EduYaleSomLib stored procedures: 
    - log_edu_yale_som_auth_webservice_error 
    - retrieve_Single_edu_yale_som_auth_User 
    - retrieve_All_edu_yale_som_auth_Roles_For_User 
    - retrieve_All_edu_yale_som_auth_Users

### SharePoint Zone Config

#### Extend the NTLM (Windows) default zone of the SharePoint content web site

*NOTE: You will keep the default zone set to NTLM (Windows) Authentication to allow the SharePoint Search service to access and index the default zone*

1. Navigate to SharePoint Central Administration > Application Management > Create or Extend Web Application
1. Click on "Extend an existing Web application"
1. Web Application: SharePoint - 80
1. Authentication provider: Negotiate (Kerberos)
1. Allow Anonymous: Yes
1. Zone: Internet

#### Configure IIS security context on the Internet zone

1. In IIS 7 Manager\Sites\[The name of your SharePoint Internet zone site]\Authentication
    1. Anonymous Authentication
        1. Click Enable
        1. Click Edit
            1. Ensure "Application Pool Identity" is selected.
    1. ASP.NET Impersonation
        1. Click Disable

*NOTE: The Internet zone uses the same application pool and process identity as the Default zone (SharePoint - 80).*

#### Set the Internet zone authentication method to FBA

1. Change the SharePoint Central Administration > Application Management > Authentication Providers
1. Web Application: SharePoint - 80
1. Select Internet zone
    1. Authentication Type: Forms 
    1. Membership provider name: mySOM [The name of the System.Web.Security.SqlMembershipProvider membership provider that you defined in web.config]
    1. Role manager name: sqlServerRoleProvider [The name of the System.Web.Security.SqlRoleProvider membership provider that you defined in web.config]
1. Perform an IISRESET
1. Internet Zone Forms Authentication Element Settings
    1. NOTE: Change `requireSSL="false"` on non-SSL instances, Recommended for **DEV ONLY**

           <authentication mode="Forms">
               <forms loginUrl="/_layouts/login.aspx" defaultUrl="default.aspx" requireSSL="true" timeout="60" slidingExpiration="true" enableCrossAppRedirects="false" name="CustomAuth" />
           </authentication>

### edu.yale.som.auth.admin Config

- Open the Visual Studio solution at: VSSolutions\edu.yale.som.auth\edu.yale.som.auth.sln
- Open edu.yale.som.auth.admin\FeatureCode\Deploy.cs
    - Modify the configuration settings to suit your environment
- Open edu.yale.som.auth.admin\12\TEMPLATE\LAYOUTS\AuthConfig.config
    - Modify the <AuthenticationMethodConfigurations> and <AuthorizationMethodConfigurations> to suit your environment

## Build

- In Visual Studio, right click on the `edu.yale.som.auth.admin` project and from the WSPBuilder menu select "Build WSP".

*NOTE: Extend any SharePoint web application zones and change them to Forms authentication before deploying and activating features so the feature web.config modifications will be added and removed properly.*

## Deploy

- Deploy the WSP to SharePoint using `stsadm`

## Activate

- In a browser, open the SharePoint - 80 (Default Zone) site.
- SharePoint Site Settings\Site Collection Administration\Site collection features, Activate the "edu.yale.som.auth.admin" feature 

*NOTE: This WSP installs all the required files to the SharePoint "12" hive and to the GAC.*

*NOTE: The Feature Receiver makes most of the required web.config changes upon Feature activation.*
  
The following files and folders are automatically deployed to each SharePoint web front end (WFE):

- `C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\12\TEMPLATE\LAYOUTS\AuthConfig.config`

*NOTE: This location allows you to globally specify the the same authentication and authorization provider configurations for all SharePoint sites. To override the global settings and specify different authentication and authorization provider configurations for a site, place AuthConfig.config in `C:\Inetpub\wwwroot\wss\VirtualDirectories\[The directory of the Internet zone of your SharePoint site]`. Then change the `<appSettings>` `Configuration_Name_Application_Auth_Config_File_Path_And_Name` value from `~/_LAYOUTS/AuthConfig.config` to `~/AuthConfig.config` in the web.config for that particular site.*

- `C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\12\TEMPLATE\LAYOUTS\login.aspx`
- `C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\12\TEMPLATE\LAYOUTS\Master_Pages\Auth.master`
- `C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\12\TEMPLATE\LAYOUTS\App_Themes\Auth_Theme\Auth_StyleSheet.css`
- `C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\12\TEMPLATE\LAYOUTS\App_Themes\Images`


## Configure Central Admin to resolve FBA users and roles

1. Ensure the following elements exist in the SharePoint Central Admin web.config:

       <webParts>
           <personalization defaultProvider="sqlServerPersonalizationProvider">
               <providers>
                   <add name="sqlServerPersonalizationProvider" connectionStringName="FBAConnectionString" applicationName="All_Users" type="System.Web.UI.WebControls.WebParts.SqlPersonalizationProvider" />
               </providers>
           </personalization>	  
       </webParts>

    <!-- -->

       <membership defaultProvider="mySOM">
           <providers>
               <add name="Domain1ADMembershipProvider" type="System.Web.Security.ActiveDirectoryMembershipProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" applicationName="All_Users" connectionStringName="Domain1ConnectionString" attributeMapUsername="sAMAccountName" />
               <add name="Domain2ADMembershipProvider" type="System.Web.Security.ActiveDirectoryMembershipProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" applicationName="All_Users" connectionStringName="Domain2ConnectionString" attributeMapUsername="sAMAccountName" />
               <add name="mySOM" type="System.Web.Security.SqlMembershipProvider" applicationName="All_Users" connectionStringName="FBAConnectionString" enablePasswordRetrieval="false" enablePasswordReset="true" requiresQuestionAndAnswer="true" requiresUniqueEmail="true" passwordFormat="Hashed" description="SQL Server membership database" minRequiredPasswordLength="8" minRequiredNonalphanumericCharacters="0" />
           </providers>
       </membership>

    <!-- -->

       <profile enabled="true" defaultProvider="sqlServerProfileProvider">
           <properties>
               <add name="FirstName" />
               <add name="LastName" />
               <add name="PreferredName" defaultValue="Not present in Profile DB" />
               <add name="WorkEmail" defaultValue="unknown@unknown.edu" />
               <add name="WorkPhone" type="System.String" defaultValue="Not present in Profile DB" />
               <add name="Office" type="System.String" defaultValue="Not present in Profile DB" />
               <add name="Department" type="System.String" defaultValue="Not present in Profile DB" />
               <add name="Title" type="System.String" defaultValue="Not present in Profile DB" />
           </properties>
           <providers>
               <clear />
               <add name="sqlServerProfileProvider" connectionStringName="FBAConnectionString" applicationName="All_Users" type="System.Web.Profile.SqlProfileProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" />
           </providers>
       </profile>

    <!-- -->

       <roleManager enabled="true" cookieProtection="All" cacheRolesInCookie="true" cookieName=".MyRolesCookie" cookieRequireSSL="true" defaultProvider="AspNetWindowsTokenRoleProvider">
           <providers>
              <add connectionStringName="FBAConnectionString" applicationName="SharePoint" description="SQL Server roles database" name="sqlServerRoleProvider" type="System.Web.Security.SqlRoleProvider" />
           </providers>
       </roleManager>

    <!-- -->

       <connectionStrings>
           <add name="ApplicationDBConnectionString" connectionString="Data Source=NORTHERNLIGHT;Initial Catalog=EduYaleSomLib;Integrated Security=SSPI;" />
           <add name="Domain1ConnectionString" connectionString="LDAP://yu.yale.edu/DC=yu,DC=yale,DC=edu" />
           <add name="Domain2ConnectionString" connectionString="LDAP://som.yale.edu/DC=som,DC=yale,DC=edu" />
           <add name="FBAConnectionString" connectionString="Data Source=NORTHERNLIGHT;Initial Catalog=Core;Integrated Security=SSPI;" />
       </connectionStrings>

## Set the Central Administration Site Application Pool Process Identity

1. Set the application pool process identity of the SharePoint Central Administration site to the [SharePoint domain service account].

*NOTE: This access allows FBA users and roles to be resolved*

## Create a project to manage FBA via the Visual Studio Website Admin Tool (WAT)

- Create a Visual Studio web project that only contains the following web.config

    *NOTE: Modify the Data Source to point to your SQL database server*

      <?xml version="1.0"?>
      <configuration>
          <connectionStrings>
              <add name='FBAConnectionString' connectionString='Data Source=[Your Database Server] ;Initial Catalog=Core;Integrated Security=SSPI;'/>	
          </connectionStrings>
          <system.web>
              <authentication mode="Forms"/>
              <roleManager enabled="true" cookieProtection="All" cacheRolesInCookie="true" cookieName=".MyRolesCookie" cookieRequireSSL="true" defaultProvider="sqlServerRoleProvider">
                  <providers>
                      <add connectionStringName="FBAConnectionString" applicationName="SharePoint" description="SQL Server roles database" name="sqlServerRoleProvider" type="System.Web.Security.SqlRoleProvider"/>
                  </providers>
              </roleManager>
              <membership defaultProvider="mySOM">
                  <providers>
                      <add name="mySOM" type="System.Web.Security.SqlMembershipProvider" applicationName="All_Users" connectionStringName="FBAConnectionString" enablePasswordRetrieval="false" enablePasswordReset="true" requiresQuestionAndAnswer="true" requiresUniqueEmail="true" passwordFormat="Hashed" description="SQL Server membership database" minRequiredPasswordLength="8" minRequiredNonalphanumericCharacters="0"/>
                  </providers>
              </membership>
          </system.web>
      </configuration>

## Add the Administrative FBA user to the web content Internet zone

1. In your WAT web project, from the Visual Studio\Website menu, select "ASP.NET Configuration" to open the Website Administration Tool (WAT).
1. Via the WAT, create a Forms Based Authentication (FBA) user in your FBA (Core) database

    *NOTE: This FBA user will have administrative access on all site collections within the Web application for all zones (Default and Internet).*

1. Create a Role called "Admins"
1. Add the FBA user to the Admins role
1. For the SharePoint content Internet zone, add a specific user with Full Control on the Web application to enable initial logon to the site collections and to perform administrative tasks.
    1. Central Administration > Application Management > Policy for Web Application >, click Add Users.
    1. Web Application: SharePoint - 80
    1. Select the Internet zone.
    1. Run the People Picker and resolve the name of the FBA user.
    1. Select Full Control and click Finish

## Add the FBA users and/or FBA groups to the web content Internet zone

1. Login to the SharePoint web content site Internet Zone as the Administrative FBA user
1. Click on Site Actions\Site Settings\Modify All Site Settings
1. In the Users and Permissions section, click People and groups
1. In the let menu, under "Groups", click More... to see all the groups
1. In each of the default SharePoint groups, add the FBA users and/or corresponding FBA groups

    Example: In the SharePoint "Owners" group for the site add the FBA "Admins" role

## Configure the Shared Services Provider (SSP) for forms-based authentication

1. Extend the NTLM default zone of the SSP site (the administration site host) to the same zone (Internet) on which the content Web application was extended
    1. SharePoint Central Administration > Application Management > Create or Extend Web Application 
    1. Click Extend an existing web application
        1. Select the Default zone of the Shared Services Provider (SSP) Web Application:
    1. Select create a new IIS web site
        1. Description: SharePoint - SharedServices1 SSP Admin Internet Zone
        1. Port: [Enter an unused port]
        1. Host Header: Leave this blank
        1. Path: C:\Inetpub\wwwroot\wss\VirtualDirectories\SharedServices1_Internet
        1. Authentication Provider: Kerberos
        1. Allow Anonymous: Yes
        1. SSL: Yes
        1. URL: https://[dns registered name for SharePoint site]:[the port you entered]
        1. Zone: '''Internet''' 
        1. Click OK

1. **Set IIS web application properties for the SharedServices1 SSP Admin Internet zone web site**
    1. Assign the SSL Cert
    1. Ensure the SSL port is set to the port you selected
1. Reset IIS to complete web site creation: Start\Run\... iisreset /noforce
1. Configure the extended Web application for forms-based authentication
    1. Central Administration > Application Management, Authentication providers
    1. Select the SSP Admin Default zone web application
    1. Click on the Internet zone link
    1. Authentication Type: Forms
    1. Membership provider name: [Enter the name of the SQL Membership provider as specified in web.config]
    1. Role manager name: [Enter the name of the SQL Role provider as specified in web.config]
    1. Change the IIS Anonymous User of the SSP Admin Internet zone IIS web application to be the same as the IIS Anonymous User of the FBA enabled Internet zone of the content Web application

        Note: This user will typically be a domain service account that is an Administrator on the SharePoint Web Front End (WFE) server and is also be the same as the user assigned as the Application Pool Identity for both the FBA enabled IIS content web application and the SSP IIS web application.  This allows the edu.yale.som.auth code to be run as the annonymous user when accessing local resources such as the AuthConfig.config file.

## Add the administrative FBA user to the SSP Internet zone

1. Ensure the web.config on each WFE for this site has the required settings as specified in the Configuration seciton of this document.
    - Allows you to resolve FBA users and groups when configuring Personalization services permissions
1. For the SSP Internet zone, add your FBA administrator user with Full Control to the Web application to enable it to access the SSP site and to perform administrative tasks.
    1. Central Administration > Application Management > Policy for Web Application 
    1. Select the SSP Admin Default zone web site
    1. Click Add Users.
    1. Select the Internet zone and click Next
    1. Run the People Picker and resolve the name of the FBA administrative user 
    1. Select Full Control
    1. Click Finish 
1. Test the SSP Internet zone FBA login at https://[Server Name]:[The port number for the Internet zone of the SharePoint SSP site]/ssp/admin

## The SharePoint Profile Import Process

edu.yale.som.auth provides methods to process users by creating a membership record for each user in the Forms Based Authentication (FBA) database including their associated roles and their profile information. These user processing methods are called during each user login when the user processing appsetting switch is enabled in web.config.  The same user processing methods are called by the edu.yale.som.auth.admin user processing webparts.
Unlike regular ASP.NET applications, SharePoint stores user profile information in its shared services provider (SSP) database instead of reading it from the FBA database.  This requires a second stage of user processing to import the FBA Profiles into SharePoint using the "SharePoint 2007 Shared Services Provider User Profile Importer" located at http://www.codeplex.com/MOSSProfileImport
Run the "SharePoint 2007 Shared Services Provider User Profile Importer" as the SharePoint System Account from a SharePoint WFE in the farm as a nightly scheduled job to keep the SSP updated.  Or you can manually run the "SharePoint 2007 Shared Services Provider User Profile Importer" after a person logs in or the single or batch FBAAdmin user processing webpart tools are run to update user profiles in SharePoint faster.

*NOTE: It takes some time after running the "SharePoint 2007 Shared Services Provider User Profile Importer" for the user profiles to propagate through SharePoint, even though the profiles do appear immediately in the SSP user priofiles.  *
  
### User Sources

- Currently all users to be processed must exist in the EduYaleSomLib database view "View_edu_yale_som_auth_Users".

    This view contains all Faculty, Students and Staff currently from the Facebook tables.  The view also includes any additional user records, such as Visitors, that are listed in the EduYaleSomLib..Non_Domain_Users table.

- Users for bulk processing are retrieved via the stored procedure EduYaleSomLib..retrieve_All_edu_yale_som_auth_Users
- Retrieval of a single user for processing is done via the stored procedure EduYaleSomLib..retrieve_Single_edu_yale_som_auth_User  

### Role Sources
- Role membership information for each Domain user (Faculty, Students and Staff) is retrieved from Active Directory (AD) security groups and mapped in AuthConfig.config to Roles in the Roles Provider (FBA Database).
- Role membership information for each non-Domain user (Visitors, etc) is retreived from the EduYaleSomLib database tables (Non_Domain_Users, Non_Domain_Users_Roles, Roles) via the stored procedure EduYaleSomLib..retrieve_All_edu_yale_som_auth_Roles_For_User

### Profile Sources
- All user profile information, properties such as FirstName, LastName, PreferredName, Email, etc., is retrieved as part of the user record in the EduYaleSomLib..View_edu_yale_som_auth_Users.
- Profile properties are declared in the web.config of the edu.yale.som.auth.webservice and in the ProfileImporter.exe.config and specified in the compiled code.  Addition or removal of profile properties requires some minor code changes.

### Error logging
- In the event that a user record is unable to be processed during bulk user processing, the error is logged to the table EduYaleSomLib..edu_yale_som_auth_webservice_Error_Log via the stored procedure EduYaleSomLib..log_edu_yale_som_auth_webservice_error
- Successful user processing is also logged in the same table.

## FBA Admin Web Parts

The following web parts can be used by "Admins" to generate FBA Membership records, profiles and roles for all or a single user in the FBA database:

1. FBAProcessAllUsers
1. FBAProcessSingleUser

### Extend the My Site Web application to the Internet zone and configure it for FBA

1. SharePoint Central Administration > Application Management > Create or Extend Web Application 
1. Click Extend an existing web application
    1. Select the Default zone of the SharePoint my site web application if you have created one
1. Enter a description
1. Port: 443
1. Enter s Host Header
1. Enter a Path
1. Authentication Provider: Kerberos
1. Allow Anonymous: Yes
1. SSL: Yes
1. URL: https://[the registered dns host name of the SharePoint mysite web site]
1. Zone: '''Internet'''
1. Click OK

### Set IIS web application properties for the Internet zone web site
1. Assign the SSL Cert
1. Ensure the SSL port is set to 443
1. Enter the IP Address registered in DNS for the mysite host name
1. Host Header: [Enter the registered dns host name of the SharePoint mysite web site]
    1. Ensure that the IP Address for the site is selected for ports 80 and 443 as this is the publicly available Internet zone.

        NOTE: Remember to assign the IP Address to the "Multiple SSL Identities for this web site" or you risk confilcts that will keep IIS web sites from starting

1. Reset IIS to complete web site creation: Start\Run\... iisreset /noforce

    You will need to set up the Service Principal Names (SPN)s for your applicaiton pool service account per the instructions in the "Create the Service Principal Name (SPN) for SharePoint Service Accounts" section in this doc.

### Set the My Site Web application Internet zone authentication method to FBA

1. Change the SharePoint Central Administration > Application Management > Authentication Providers settings for the My Site Internet zone:
    1. Authentication Type: Forms 
    1. Membership provider name: [Enter the SQL Membership Provider as specified in web.config]
    1. Role manager name: [Enter the SQL Roles Provider as specified in web.config]

### My Site web.config

1. Ensure the "my site" web.config on each WFE has the required settings as specified in the Configuration seciton of this document.

    **Note: This web.config should have had the defaultUrl="/Pages/default.aspx" attribute removed from the <forms> element in the <authentication mode="Forms"> element.  This was required to allow the redirect back from CAS (with a ticket) to be properly redirected to create the my site.**

### Enable forms-based authenticated users to edit personalization services permissions

1. Grant the FBA administrative user rights to add permissions for personalization by logging in to the SSP administration site Default zone as a the [SharePoint domain service account].
1. Click on Personalization services permissions
1. Click Add Users/Groups
1. Resolve the FBA administrative group "Admins" to allow the FBA admin users to set personalization services permissions for other FBA users in the SSP site Internet zone

        Example, Admins

1. Grant these FBA administrators all of the following permissions:
    - Create Personal Site: This permission is required to make the My Site link visible, and enables users to create a My Site
    - Use Personal Features: This permission enables users to access SSP and My Site features.
    - Manage user profiles: This permission enables users to view and manage user profiles from the Profile Store.
    - Manage Audiences: This permission enables users to manage audiences.
    - Manage Permissions: This permission enables permission management on an SSP.
    - Manage Usage Analytics: This permission enables users to manage and configure usage analysis. 

### Change the SSP Internet zone to point to the new My site Internet zone

1. Login into the SSP Internet zone administration Web site using forms-based authentication
1. Click on My Site Settings

    1. Personal site provider: [the URL to the SharePoint mysite Internet zone]
    1. Click OK

1. Change the IIS Anonymous User of the My Site Internet zone IIS web application to be the same as the IIS Anonymous User of the FBA enabled Internet zone of the content Web application

## Grant forms-based authenticated users rights to create "my sites"

1. As an FBA Admin, login to the Shared Services Administration Internet Zone
1. Click on Personalization services permissions
1. Click Add Users/Groups
1. Resolve the FBA users and/or the FBA roles that you want to set personalization services permissions for:
    1. For those FBA users\roles that you want to create a my site, grant them the "Personal Site" permission. example: sqlserverroleprovider:all_portal_users 
1. Central Administration > Application Management > Policy for Web Application
    1. Select the My Site default zone web application
    1. Click Add Users
    1. Select the '''Internet''' zone and click Next
    1. Run the People Picker and resolve the FBA users and/or the FBA roles that you want to allow to create My Sites.  For example: [Roles Provider name]:Admins or [Roles Provider name]:All_Portal_Users
    1. Add those FBA users and/or FBA Roles with '''Full Read''' access
1. Log in to the Internet zone of the web content application using one of those non-admin mysite enabled FBA users to make sure the Welcome control displays the My Site link
1. Click on the My Site link to create a my site for that FBA user
1. Log in to the mysite web site as the same FBA user to create your mysite
1. Remember to modifiy the server host and network firewalls if you used a new port for the My Site Internet zone

### Personalizing Web Parts

1. As an FBA Admin, login to the SSP Internet Zone
1. Click on Personalization services permissions
1. Click Add Users/Groups
1. Resolve the FBA users and/or FBA roles you want to set personalization services permissions for:
1. Grant the FBA user or FBA roles, example, [Roles Provider name]:all_portal_users, the "Use Personal Features" permission
1. Login to the content site Internet zone as a site collection admin, create a SharePoint Group, if it doesn't already exist, called "All_Portal_Users" in the site collection with no permission level
1. Add [Roles Provider name]:All_Portal_Users Role to the new SharePoint Group called "All_Portal_Users"
1. In the left hand navigation, click on "Site Permissions"
1. In the top navigation of the center column, select Settings\Permission Levels
1. Click "Add a Permission Level"
    1. Name: All_Portal_Users
1. Check the option under "Personal Permissions" called "Update Personal Web Parts  -  Update Web Parts to display personalized information"
    - This will automatically check any other required permissions
1. Click Create
1. Go back to SharePoint groups and click the edit icon for the SharePoint Group "All_Portal_Users"
1. Check the Permission Level "All_Portal_Users" if not aleady checked
1. Click OK

- Note: if you use FBA groups, you must assign your FBA users to those FBA groups for the My Site link to be visible for them.
    - Non-FBA, e.g. Active Directory Domain, users can be automatically added to FBA groups upon their logging in to the FBA configured Internet Zone. This is acomplished by adding Role/Group mappings to an authentication method's setup in AuthConfig.config
- The FBA Role sqlserverroleprovider:All_Portal_Users is Mapped to the Active Directory (AD) Group [domain]\SP_All_Portal_Users
- To grant non-AD mapped FBA Roles, such as [Roles Provider name]:Visitors, these personalization features perform the same steps but replace:
    - The "All_Portal_Users" SharePoint group with the "Visitors" SharePoint group
    - The "All_Portal_Users" permission level with the "Read" permission level
    - The "[Roles Provider name]:All_Portal_Users" role with the "[Roles Provider name]:Visitors" role

<!-- WORK FROM HERE -->

= Configuration Examples =

* NOTE: These configuration settings are provided as examples of a system configured to use the authentication methods: Visitor (Forms Based Authentication), DOMAIN1 (Central Authentication Service -CAS | Active Directory Domain1), DOMAIN2 (Active Directory Domain).  You will need to adapt these configurations to meet the needs of your implementation.  You will also need to refactor the resource reference names in the edu.yale.som.auth.dll references.cs and rebuild the solution if you make changes to the connection string names, appsetting names, etc.

== The following additional settings need to be included in the web.configs of your Forms Based Authentication (FBA) enabled SharePoint MySite Zones. ==
* Remove the defaultUrl="/Pages/default.aspx" property from the <authentication mode="Forms"> element.  This is required to allow the redirect back from CAS (with a ticket) to be properly redirected to create the my site.

== The following settings need to be included in the web.configs of your SharePoint Central Admin Sites ==
* Uses AspNetWindowsTokenRoleProvider as the default role provider
** Allows you to still login to the Central Admin site with Windows authentication but also see the FBA providers when using the people picker to grant FBA users permissions in SharePoint.
<pre>
	<connectionStrings>
		<add name="Domain1ConnectionString" connectionString="LDAP://[ENTER YOUR LDAP DOMAIN]/DC=[ENTER],DC=[YOUR],DC=[DOMAIN]"/>
		<add name="Domain2ConnectionString" connectionString="LDAP://[ENTER YOUR LDAP DOMAIN]/DC=[ENTER],DC=[YOUR],DC=[DOMAIN]"/>
		<add name="FBAConnectionString" connectionString="Data Source=[Enter the name of your database server];Initial Catalog=Core;Integrated Security=SSPI;"/>
		<add name="ApplicationDBConnectionString" connectionString="Data Source=[Enter the name of your database server];Initial Catalog=EduYaleSomLib;Integrated Security=SSPI;"/>
	</connectionStrings>
</pre>

<pre>
    <roleManager enabled="true" cookieProtection="All" cacheRolesInCookie="true" cookieName=".MyRolesCookie" cookieRequireSSL="true" defaultProvider="AspNetWindowsTokenRoleProvider">
			<providers>
				<add connectionStringName="FBAConnectionString" applicationName="SharePoint" description="SQL Server roles database" name="sqlServerRoleProvider" type="System.Web.Security.SqlRoleProvider"/>
			</providers>
		</roleManager>
		<membership defaultProvider="FBAMembershipProvider">
			<providers>
				<add name="Domain1ADMembershipProvider" type="System.Web.Security.ActiveDirectoryMembershipProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" applicationName="All_Users" connectionStringName="Domain1ConnectionString" attributeMapUsername="sAMAccountName"/>
				<add name="Domain2ADMembershipProvider" type="System.Web.Security.ActiveDirectoryMembershipProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" applicationName="All_Users" connectionStringName="Domain2ConnectionString" attributeMapUsername="sAMAccountName"/>
				<add name="FBAMembershipProvider" type="System.Web.Security.SqlMembershipProvider" applicationName="All_Users" connectionStringName="FBAConnectionString" enablePasswordRetrieval="false" enablePasswordReset="true" requiresQuestionAndAnswer="true" requiresUniqueEmail="true" passwordFormat="Hashed" description="SQL Server membership database" minRequiredPasswordLength="8" minRequiredNonalphanumericCharacters="0"/>
			</providers>
		</membership>
		<webParts>
			<personalization defaultProvider="sqlServerPersonalizationProvider">
				<providers>
					<add name="sqlServerPersonalizationProvider" connectionStringName="FBAConnectionString" applicationName="All_Users" type="System.Web.UI.WebControls.WebParts.SqlPersonalizationProvider"/>
				</providers>
			</personalization>
		</webParts>
		<profile enabled="true" defaultProvider="sqlServerProfileProvider">
			<properties>
				<add name="FirstName"/>
				<add name="LastName"/>
				<add name="PreferredName" defaultValue="Not present in Profile DB"/>
				<add name="WorkEmail" defaultValue="unknown@unknown.edu"/>
				<add name="WorkPhone" type="System.String" defaultValue="Not present in Profile DB"/>
				<add name="Office" type="System.String" defaultValue="Not present in Profile DB"/>
				<add name="Department" type="System.String" defaultValue="Not present in Profile DB"/>
				<add name="Title" type="System.String" defaultValue="Not present in Profile DB"/>
			</properties>
			<providers>
				<clear/>
				<add name="sqlServerProfileProvider" connectionStringName="FBAConnectionString" applicationName="All_Users" type="System.Web.Profile.SqlProfileProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"/>
			</providers>
		</profile>
</pre>

== The following settings need to be included in the web.configs of your NTLM enabled SharePoint Site Shared Service Provider (SSP) Default Zones. ==
* Allows you to resolve FBA users and groups when configuring Personalization services permissions
* Update the connection strings
<pre>
	<connectionStrings>
		<add name="Domain1ConnectionString" connectionString="LDAP://[ENTER YOUR LDAP DOMAIN]/DC=[ENTER],DC=[YOUR],DC=[DOMAIN]"/>
		<add name="Domain2ConnectionString" connectionString="LDAP://[ENTER YOUR LDAP DOMAIN]/DC=[ENTER],DC=[YOUR],DC=[DOMAIN]"/>
		<add name="FBAConnectionString" connectionString="Data Source=[Enter the name of your database server];Initial Catalog=Core;Integrated Security=SSPI;"/>
		<add name="ApplicationDBConnectionString" connectionString="Data Source=[Enter the name of your database server];Initial Catalog=EduYaleSomLib;Integrated Security=SSPI;"/>
	</connectionStrings>
</pre>

* Update the providers
<pre>
		<roleManager enabled="true" cookieProtection="All" cacheRolesInCookie="true" cookieName=".MyRolesCookie" cookieRequireSSL="true" defaultProvider="sqlServerRoleProvider">
			<providers>
				<add connectionStringName="FBAConnectionString" applicationName="SharePoint" description="SQL Server roles database" name="sqlServerRoleProvider" type="System.Web.Security.SqlRoleProvider"/>
			</providers>
		</roleManager>
		<membership defaultProvider="FBAMembershipProvider">
			<providers>
				<add name="Domain1ADMembershipProvider" type="System.Web.Security.ActiveDirectoryMembershipProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" applicationName="All_Users" connectionStringName="Domain1ConnectionString" attributeMapUsername="sAMAccountName"/>
				<add name="Domain2ADMembershipProvider" type="System.Web.Security.ActiveDirectoryMembershipProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" applicationName="All_Users" connectionStringName="Domain2ConnectionString" attributeMapUsername="sAMAccountName"/>
				<add name="FBAMembershipProvider" type="System.Web.Security.SqlMembershipProvider" applicationName="All_Users" connectionStringName="FBAConnectionString" enablePasswordRetrieval="false" enablePasswordReset="true" requiresQuestionAndAnswer="true" requiresUniqueEmail="true" passwordFormat="Hashed" description="SQL Server membership database" minRequiredPasswordLength="8" minRequiredNonalphanumericCharacters="0"/>
			</providers>
		</membership>
		<webParts>
			<personalization defaultProvider="sqlServerPersonalizationProvider">
				<providers>
					<add name="sqlServerPersonalizationProvider" connectionStringName="FBAConnectionString" applicationName="All_Users" type="System.Web.UI.WebControls.WebParts.SqlPersonalizationProvider"/>
				</providers>
			</personalization>
		</webParts>
		<profile enabled="true" defaultProvider="sqlServerProfileProvider">
			<properties>
				<add name="FirstName"/>
				<add name="LastName"/>
				<add name="PreferredName" defaultValue="Not present in Profile DB"/>
				<add name="WorkEmail" defaultValue="unknown@unknown.edu"/>
				<add name="WorkPhone" type="System.String" defaultValue="Not present in Profile DB"/>
				<add name="Office" type="System.String" defaultValue="Not present in Profile DB"/>
				<add name="Department" type="System.String" defaultValue="Not present in Profile DB"/>
				<add name="Title" type="System.String" defaultValue="Not present in Profile DB"/>
			</properties>
			<providers>
				<clear/>
				<add name="sqlServerProfileProvider" connectionStringName="FBAConnectionString" applicationName="All_Users" type="System.Web.Profile.SqlProfileProvider, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"/>
			</providers>
		</profile>
</pre>

== General AuthConfig.config Settings for all edu.yale.som.auth Forms Based Authentication (FBA) enabled sites ==
# Defines the authentication and authorization provider properties to be deserialized from xml into temporary objects and used by the web application to initialize the IAuthenticate and IAuthorize implementation objects.
# Data Dictionary:
## <AuthenticationMethodConfigurations> = REQUIRED The collection of authentication methods
### <AuthenticationMethodConfig> = REQUIRED Contains the properties of a single authentication method
#### <_classTypeName>= REQUIRED The name of the class that implements IAuthenticate for this authentication method
#### <_activeDirectoryBindPath> = OPTIONAL The LDAP bind path to the search root of the Active Directory domain that the Membership Provider for this authentication method uses
#### <_domainName> = OPTIONAL The Active Directory domain name that the Membership Provider for this authentication method uses
#### <_membershipProviderName> = REQUIRED The name of the ASP.NET 2.0 Membership provider, defined in web.config, that this authentication method will use
#### <_displayName> = REQUIRED The text to assign to the ASP.NET web control displayed in login.aspx that users select to use this authentication method
#### <_id> = REQUIRED The unique ID property used to programmatically reference this authentication method. Used as the unique ID property of the ASP.NET web control displayed in login.aspx which must be different from the ID of any other web control displayed with the login page
#### <_authorizationMethodClassTypeName> = REQUIRED The name of the class that implements IAuthorize for the authorization method that must be associated with this authentication method. 
#### <_loginHelpContent> = OPTIONAL The text to display on the login help panel of the login page.  HTML can be inserted in XML using CDATA.
#### <_authenticationApplicationCollection> = REQUIRED The collection of Membership Provider applications  
##### <AuthenticationApplication> = OPTIONAL Contains the properties that pertain to a single Membership Provider "application"
###### <_authenticationApplicationName> = REQUIRED The name of the Membership Provider application property as defined in web.config
###### <_roleMapCollection> = REQUIRED The collection of mappings between roles and groups for this Membership Provider application
####### <RoleMap> = OPTIONAL Contains the role and group for a single mapping
######## <_roleIdentifier> = REQUIRED The name of the FBA role in the FBA Role Provider
######## <_groupIdentifier> = REQUIRED The distinguished name of the AD group in the AD Membership Provider
######## <_groupName> = REQUIRED The (domain\group) name of the AD group in the AD Membership Provider
### <AuthorizationMethodConfigurations> = REQUIRED The collection of authorization methods 
#### <AuthorizationMethodConfig> = REQUIRED Contains the properties of a single authorization method
##### <_classTypeName> = REQUIRED The name of the class that implements IAuthorize for this authorization method
##### <_authorizatonRoleManagerProvider> = REQUIRED The name of the ASP.NET 2.0 Role provider, defined in web.config, that this authorization method will use
##### <_authorizatonMembershipProvider> = OPTIONAL The name of the ASP.NET 2.0 Membership provider, defined in web.config, that this authorization method uses to assign roles to Membership Users
##### <_id> =  REQUIRED The unique ID property used to programmatically reference this authorization method 

= Basic Application Programmer Index (API) for edu.yale.som.auth =
       
== SAMPLE: Checking role membership and retrieving membership properties for the current membership user ==
<pre>
using System;
using System.Web;
using System.Web.Security;
using System.Web.UI;
using System.Web.UI.WebControls;
using edu.yale.som.auth;

public partial class _default : System.Web.UI.Page
{
    protected void Page_Load(object sender, EventArgs e)
    {
        if (!this.IsPostBack)
        {
            // Check user properties from the user identity generated from Forms Based Authentication
            bool isAuth = HttpContext.Current.User.Identity.IsAuthenticated;
            string authType = User.Identity.AuthenticationType;
            string name = User.Identity.Name;
            bool isAdmin = User.IsInRole("Admins");
            Response.Write("User Authenticated: " + isAuth + "<br />");
            Response.Write("Authentication Type: " + authType + "<br />");
            Response.Write("User Name: " + name + "<br />");
            Response.Write("Admin User: " + isAdmin + "<br />");

            // NOTE: The values in the User Identity, e.g. user name, may be different than the values in the selected Membership Provider
            // Example: The end user selects an authentication method that authenticates against Active Directory but authorizes against a SQL Server FBA Data Store.
            // Active Directory user names (cannonical name, cn=) do not include the "domain\".
            // The SQL Server FBA Data Store may add the "domain\" to it's user names to distinguish them from non-domain FBA users.
            // See the examples below on edu.yale.som.auth.AuthClientUtil for how to obtain the Membership User values from the selected Membership Provider instead of the
            // FBA authenticated User.Identity or the Default Membership and Role Providers.

            // Instantiate the utility object
            edu.yale.som.auth.AuthClientUtil authClientUtil = new edu.yale.som.auth.AuthClientUtil();
            // Get the currently logged in user from the SELECTED Membership Provider
            MembershipProvider selectedMembershipProvider = authClientUtil.getSelectedMembershipProvider();
            MembershipUser currentMembershipUser = selectedMembershipProvider.GetUser((string)Session["Session_Variable_Name_Current_Authenticated_User_Name"], false);

            // Check if a membership user was retrieved
            if (currentMembershipUser != null)
            {
                // Retrieve the membership user's email address from the SELECTED Membership Provider
                string emailAddress = currentMembershipUser.Email;
                EmailAddressLabel.Text = emailAddress;
                // Check for a certain role in the SELECTED Role Provider
                if (authClientUtil.getSelectedRoleProvider().IsUserInRole((string)Session["Session_Variable_Name_Current_Authenticated_User_Name_In_Role"], "Admins"))
                {
                    // User is a member of the role
                    RolesBulletedList.Items.Add("Admins");
                }
                if (authClientUtil.getSelectedRoleProvider().IsUserInRole((string)Session["Session_Variable_Name_Current_Authenticated_User_Name_In_Role"], "Visitors"))
                {
                    // User is a member of the role
                    RolesBulletedList.Items.Add("Visitors");
                }
            }
        }
    }
}
</pre>

== Adding new Authentication or Authorization providers ==

=== Authentication ===
# In the edu.yale.som.auth class library project, create a new class named [authentication type name]Authenticate
# Declare your new class to implement the IAuthenticate interface

=== Authorization ===
# In the edu.yale.som.auth class library project, create a new class named [authorization type name]Authorize
# Declare your new class to implement the IAuthorize interface

== Adding a new Application type ==
# In the edu.yale.som.auth class library project, create a new class named Auth[application type name]
# Declare your new class to implement the IAuth interface
