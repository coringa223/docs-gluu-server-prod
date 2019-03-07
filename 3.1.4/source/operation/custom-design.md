# Customizing Public Pages

Most organizations will want to edit and customize the look and feel of public-facing Gluu Server pages to match their own corporate branding. 
The following documentation provides the file locations of public facing pages, as well as instructions for adding custom HTML, CSS, and Javascript files to your Gluu Server. 

Public pages include:  

  - All included sign-in pages  
  - Registration  
  - Password Recovery  
  - Error Pages  

!!! Warning 
    Customizations should only be made by people with a solid understanding of web development. Before changing any files, we recommend creating backups to easily revert your instance to its original state.
    
<!--
 This section has been removed, since an updated WAR is overwritten when the Gluu Server is upgraded, and a newer and more consistent method has been added below
## Overview
  
The Gluu Server's public facing pages are `xhtml` files. Each Gluu Server component is deployed as a separate archive in WAR format. When any component's service is started, its archive is unpacked ("exploded") to Jetty's temporary file directory located under `/opt/jetty-<VERSION>/temp/` before it'll be able to start serving requests for associated functionality. 

To customize any files used by a component, they need to be changed either at that temporary location, or inside the corresponding archive itself. Note that changes made directly to unpacked files under `/opt/jetty-<VERSION>/temp/` won't be persisted--each time a component's service is restarted its WAR archive will be re-exploded, overwriting the existing content on the disk.

A typical example would be customizing oxAuth's login page. There are two ways to achieve this:

1. Put changed files under `/opt/gluu/jetty/oxauth/custom/` directory, so they could be used instead of the standard files in `oxauth.war`. (Note: the same approach will work for oxTrust if files are placed under `/opt/gluu/jetty/identity/custom/`). The benefit of using this method is that your customizations won't be disturbed by any changes to `oxauth.war` or `identity.war` later on (for example, in case this Gluu instance will be patched or updated, and a component's WAR archive will get overwritten). More on this method below. 

1. Unpack the needed files from `/opt/gluu/jetty/oxauth/webapps/oxauth.war` with a tool like `jar`, update them and add them back to the archive with all required dependencies (**not recommended**);

-->

## Directory structure and mappings

!!! Note
    Log in to the Gluu Server chroot before working on design customizations for any pages.

New directories trees have been added inside the Gluu Server `chroot` to make page customizations easier. 
Each such tree is placed in the configuration directory of the corresponding Gluu component (only 
customization of oxAuth and oxTrust pages is supported at the moment by this feature). 
The new directory structure can be illustrated as follows (only directories related to this feature are shown for clarity):

### oxAuth

```
/opt/gluu/jetty/oxauth/
|-- custom
|   |-- i18n
|   |-- libs
|   |-- pages
|   `-- static
```

### oxTrust

```
/opt/gluu/jetty/identity/
|-- custom
|   |-- i18n
|   |-- libs
|   |-- pages
|   `-- static
```

### Subdirectories 
Customized `i18n` should be placed in the following directories:
```
/opt/gluu/jetty/identity/custom/i18n
/opt/gluu/jetty/oxauth/custom/i18n
```
Resources from this folder will be loaded at next service restart.

Sub-directories `custom/pages` have a special purpose. They enable overriding exploded `xhtml` pages from the unpacked WAR archive. The path to exploded war conforms to following scheme:

```
/opt/jetty-<VERSION>/temp/jetty-localhost-<PORT_NUMBER>-<COMPONENT_NAME>.war-_<COMPONENT_NAME>-any-<RANDOM_TAG>.dir/webapp/
```

So, for example, the path to an exploded oxAuth's WAR archive directory may look like this 
(and may be changed the next time the corresponding service is restarted):

```
/opt/jetty-9.3/temp/jetty-localhost-8081-oxauth.war-_oxauth-any-9071517269463235631.dir/webapp/
```

Thus, a modified `login.xhtml` page put under `custom/pages/` will be used instead of 
`webapp/login.xhtml` file from the exploded archive. You can use files unpacked there 
as a base for your own customized files.

!!! Warning 
    Jetty included in earlier Gluu 3.x packages is known to create duplicated 
    directories under `/opt/jetty-<VERSION>/temp/` for each of its components. 
    In case of encountering this issue, it's recommended to stop corresponding 
    service and remove all subdirectories related to it from the `temp/` 
    directory. After starting service again its WAR archive will be unpacked there again.

Customized `libs` for oxAuth to use should be placed in the following directories:
```
/opt/gluu/jetty/identity/custom/libs
/opt/gluu/jetty/oxauth/custom/libs
```

Custom CSS or images should be placed under `custom/static` directory. To avoid 
collisions with static resources from WAR files, Gluu maps this folder 
to URL's path like this: `/{oxauth|identity}/ext/resources`

So, for example, CSS file placed at this path:

```
/opt/gluu/jetty/oxauth/custom/static/stylesheet/theme.css
```

...will be externally available at a URL similar to this:

```
https://your.gluu.host/oxauth/ext/resources/stylesheet/theme.css
```

...and should be referenced from inside of source codes of customized files by path like this:

```
/oxauth/ext/resources/stylesheet/theme.css
```

All images should be placed under: 

`/opt/gluu/jetty/oxauth/custom/static/img`

!!! Note
    You can change the logo on every public-facing page here. Place your image in `/static/img` and name it `logo.png`.

And all CSS are inside:

`/opt/gluu/jetty/oxauth/custom/static/stylesheet`

## Location of key webpage source files

Default Gluu's public-facing pages can be a good base for your organization's customized 
ones. Aside from extracting them directly from a corresponding WAR file, they can be found 
at Jetty's temp directory to which they are unpacked each time a corresponding service starts.

### oxAuth

oxAuth is the core Gluu CE component, handling all authentication in the framework and implementing OpenID Connect and UMA flows. Most of the web UI pages displayed to end users belong to oxAuth (login/logout/authorization flows).

Base directory:
`/opt/jetty-<VERSION>/temp/jetty-localhost-8081-oxauth.war-_oxauth-any-<RANDOM_TAG>.dir/webapp/`

- Default login page:
    `./login.xhtml`
- Authorization page:
    `./authorize.xhtml`
- Logout page:
    `./logout.xhtml`
- Error page:
    `./error.xhtml`
- Custom authentication scripts; XHTML files under:
    `./webapp/auth/`

### oxTrust:

oxTrust is responsible for displaying the Gluu Server's default registration page, as well as the administrator web UI's pages. 

Base directory:
`/opt/jetty-<VERSION>/temp/jetty-localhost-8082-identity.war-_identity-any-<RANDOM_TAG>.dir/webapp/`

- Registration page:
    `./register.xhtml`

## Applying changes

The oxAuth and oxTrust services need to be restarted for customizations to be applied. The next commands will restart corresponding Jetty's JVMs inside container:

```
# service oxauth stop
# service oxauth start
# service identity stop
# service identity start
```

!!! Note
    There is a 10 second delay on page modification reload.

## An Example: Removing the Gluu copyright 

For a good practical example, let's consider a task of removing the Gluu copyright 
at the bottom of oxAuth's login page. You can follow these steps to achieve this:

1. Log into the Gluu container: `# service gluu-server-3.1.4 login`

2. Create a new directory structure under `custom/pages/` to accomodate new customized page: `# mkdir -p /opt/gluu/jetty/oxauth/custom/pages/WEB-INF/incl/layout/`

3. Get a default template page from the exploded WAR archive and put it in the path under `custom/pages` directory, which will allow it to override the original page (your path to the exploded WAR will differ from the one used here): `# cp /opt/jetty-9.3/temp/jetty-localhost-8081-oxauth.war-_oxauth-any-9071517269463235631.dir/webapp/WEB-INF/incl/layout/template.xhtml /opt/gluu/jetty/oxauth/custom/pages/WEB-INF/incl/layout/template.xhtml`

4. Modify the new file by removing or editing the following snippet in it:

    ```
      <s:fragment rendered="#{not isLogin}">
          <div class="footer">
              <p>Copyright <a href="http://www.gluu.org">Gluu</a> All rights reserved.</p>
          </div>
      </s:fragment>
    ```

5. Assign appropriate permissions to new directories and files: `# chown -R jetty:jetty /opt/gluu/jetty/oxauth/custom/pages/ && chmod -R a-x+rX /opt/gluu/jetty/oxauth/custom/pages/`

You may opt to copy the default oxAuth login page (`login.xhtml`) to the custom files 
directory as well, and add some customizations to it:

    ```
      # cp /opt/jetty-9.3/temp/jetty-localhost-8081-oxauth.war-_oxauth-any-9071517269463235631.dir/webapp/login.xhtml   /opt/gluu/jetty/oxauth/custom/pages/
    ```

Don't forget to apply appropriate file system permissions if needed.
Restarting oxAuth's service inside container will display the changes: `# service oxauth stop && service oxauth start`