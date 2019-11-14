# Guest user access

It is possible to allow public access to some living applications (without the need to sign in to Bonita).  

When accessing a public application without any active session on Bonita platform the user will be automatically logged in with a guest user account.

:::info 
**Note:** For Enterprise, Performance, Efficiency, and Teamwork editions only.
:::

::: warning  
 This feature is deactivated by default. When you activate it, make sure you give limited access rights to the guest user account (Eg.: use a [dedicated profile](#prerequisite) instead of the default provided ones and make sure the pages you put in the application don't grant more access rights to the REST API than what you accept to grant).  
 For better security it is recommended when using this feature, to have a reverse proxy or a load balancer configured to prevent too many requests to the REST API to be performed, consuming all the platform resources (Distributed denial of service attacks, etc...).
:::

::: warning  
 This feature is different from the auto-login feature that was available in the 6.x versions of Bonita BPM as it doesn't give directly access to a process form publicly. It is intended for living applications access.
:::

<a id="prerequisite"/>

## Guest profile and dedicated user account

+ In order to use this feature, you need to create a new user account in Bonita organisation. This account will be the one used to log in automatically a user that tries to access a public living application if he is not logged in to Bonita platform yet.

+ You also need to create a new custom profile and add the user account created in the previous step as a member of this profile. This profile will be the one to use as the profile of the living applications that require public access. If an application should also be accessible by users already logged in with their own bonita account, then they should also be members of this profile.  

::: warning  
 You should not use any of the default profiles of Bonita portal as profiles for your public living applications because, by doing that, you would give access to the guest user account to those portal profiles (and their priviledges).
:::

:::info 
**Note:** Since the applications will be public, it is recommended to have a group or a role containing all the users of Bonita platform organisation and add this group or role as a member of the profile used for public application (otherwise users already logged in with their account will get a 403 error when trying to access the public applications).
:::

## Configure Bonita to allow guest user access to some applications

The bundle already contains the files needed to setup guest user access with Bonita platform.
To activate it:

1.  Use the [platform setup tool](BonitaBPM_platform_setup) to retrieve the current configuration (Eg. setup.sh/.bat pull). You need to execute the following actions in the folder of the tenant in which the applications which requires public access are deployed.

2. In the tenant_portal folder of the target tenant: `<BUNDLE_HOME>/setup/platform_conf/current/tenants/<TENANT_ID>/tenant_portal`,
   update the file `authenticationManager-config.properties` as follows:
    ```
            [...]
            # logout.link.hidden=true
       -->  auth.tenant.guest.active=true
       -->  auth.tenant.guest.username=guest
       -->  auth.tenant.guest.password=guest
       -->  auth.tenant.guest.apps=[public,guest] 
    ```
    
    Make sure to set the username and password of the guest user account created in the section [Guest profile and dedicated user account](#prerequisite).
    The property "auth.tenant.guest.apps" contains the list of URL tokens of the applications that require public access (in this example, "public" and "guest"). Replace it with your applications tokens

3. Use the [platform setup tool](BonitaBPM_platform_setup) again to save your changes into the database (Eg. setup.sh/.bat push).  
   Restart Bonita server.

4. If your configuration is correct you should be able to access your public application directly with the URL `http://<host>:<port>/bonita/apps/<app token>` without being logged in first.  
   You are done.

## Starting a process as guest user

You may require your public living application to provide a link to start a process instance (case). In order for the process instantiation form to be displayed to a guest in an application, what you need to do is:
- add the guest user account in the actor mapping of the actor which is defined as [initiator of the process](actors#toc1)
- make sure the REST API authorization mechanism allows the guest user to start the process instance.  

In order for the authorization filter to let the guest user make the necessary API requests, the best solution is to activate the [dynamic authorisation checking](rest-api-authorization#dynamic_authorization) for those requests :  

1.  Use the [platform setup tool](BonitaBPM_platform_setup) to retrieve the current configuration (Eg. setup.sh/.bat pull). You need to execute the following actions in the folder of the tenant in which the applications which requires public access are deployed.

2. In the tenant_portal folder of the target tenant: `<BUNDLE_HOME>/setup/platform_conf/current/tenants/<TENANT_ID>/tenant_portal`,
   update the file `dynamic-permissions-checks-custom.properties` to uncomment 3 lines as follows (they may already be uncommented if you use the dynamic authorisation checking instead of the default static authorization checking) :
    ```
        [...]
        ## ProcessPermissionRule
        ## Let the user do get only on processes he deployed or that he supervised
    --> GET|bpm/process=[profile|Administrator, check|org.bonitasoft.permissions.ProcessPermissionRule]
        #POST|bpm/process=[profile|Administrator, check|org.bonitasoft.permissions.ProcessPermissionRule]
        #PUT|bpm/process=[profile|Administrator, check|org.bonitasoft.permissions.ProcessPermissionRule]
        #DELETE|bpm/process=[profile|Administrator, check|org.bonitasoft.permissions.ProcessPermissionRule]
    --> GET|bpm/process/*/contract=[profile|Administrator, check|org.bonitasoft.permissions.ProcessPermissionRule]
    --> POST|bpm/process/*/instantiation=[profile|Administrator, check|org.bonitasoft.permissions.ProcessInstantiationPermissionRule]
        [...]
    ```
:::info 
**Note:** This modification will activate the [dynamic authorisation checking](rest-api-authorization#dynamic_authorization) instead of the default static authorization checking for the following request : `GET|bpm/process`, `GET|bpm/process/*/instantiation`, `POST|bpm/process/*/instantiation`. It does not impact only the guest user access to these resources but the whole organization.  
When a user tries to get the contract or general information of a process it will make sure he is either:
- a member of the actor mapping of the process
- an administrator 
- or a manager of the process

When someone tries to start a process, it will make sure he is either:
- a member of the actor mapping defined as the initiator of the process
- an administrator
- or a manager of the process
:::

If in your process instantiation form you make other requests to Bonita REST API, you may need to uncomment more lines in `dynamic-permissions-checks-custom.properties` accordingly.

3. Use the [platform setup tool](BonitaBPM_platform_setup) again to save your changes into the database (Eg. setup.sh/.bat push).  
   Restart Bonita server.

4. If your configuration is correct a guest user should be able to start a process instance (case) without being logged in first.  
   You are done.

## Login behaviour

The default Bonita layout handles the guest user account by providing a "Sign in" link instead of the user modal link in the header.
