# Sys Admin Crumbs #


This is a collection of useful (atleast to me) snippets and recipes. It is a work in progress so may not contain a proper structure at this stage.


## Sudo ##


### How to Add groups/users to `/etc/sudoers`? ###

This is the recommended approach as it avoides polluting the original sudo file and makes management & maintenance easy.

Instead of modifying the `/etc/sudoers` file do the following:

1. Create a `/etc/sudoers.d` dir if it does not exist
2. Create a file with no extension and file permission 0440

```
chmod 0440 <filename>
chown root:root <filename>
```

3. Add the file to `/etc/sudoers.d/` dir. 

4. Include the directory in the sudoers file by performing:
   
   ```
   # /usr/sbin/visudo
   sudo visudo

   # Add the following to the sudoers file. Note the hash has been explicitly added:
   #includedir /etc/sudoers.d 

   ```
4. All files now added to `/etc/sudoers.d` will be automatically imported 


### How to check what sudo acces a user has? ###

`sudo -l -U bob`



## SSH ##


### How to copy your key to a remote server? ###

`ssh-copy-id -i username@remoteserver`

ssh-copy-id is a script that uses ssh to log into a remote machine to append your key to the authorized_keys file and sets the correct permission on `~/.ssh` and `~/.ssh/authorized_keys` file.

The '-i' identity option gives it the identify file  (defaults to `~/.ssh/id_rsa.pub`)

## Building RPMs ##

The following guide focuses on how source files can be retrofitted to be installed as RPMs.

### Basics of Building RPMs ###

 Do all of the following as a **non-root** user!

Create the directory tree like the following:

```
    # Create a working dir
    mkdir rpmbuild
    cd rpmbuild
    # Mandatory directories to build the RPM
    mkdir -p {BUILD,SPECS,SOURCES,RPMS,SRPMS}
```


Specify the build area:

```
# Add the following line to ~/.rpmmacros (create it 
# if it doesn't exist)
%_topdir (echo $HOME)/rpmbuild
```


#### Macros ####

The `~/.rpmmacros` file contains global macros defined as `%{_macroname}`. Notice the global macro birthmark of `_` preceding the  name.  

Local macros defined in the *spec* file take the form of `%{macroname}`

Usually in the *preamble* section of a spec file you will see something like this:

```
Name:           test
Version:        1.0
Release:        0
...
```
The above fields can then later be referenced as:
```
# Source0: test-1.0.zip
Source0:        %{name}-%{version}.zip
```


##### Examples of built-in macros #####
```
%_prefix /usr
%_sysconfdir /etc
%_localstatedir /var
%_infodir /usr/share/info
%_mandir /usr/share/man
%_initrddir %{_sysconfdir}/rc.d/init.d
%_defaultdocdir %{_usr}/share/doc
```

##### How to define custom macros? #####

In addition to the built-in macros, you can define your own to make it easier to manage your packages.

```
#%define macro_name value
%define major 2

# Accessing macro
%{major}
```

#### Deep Dive ####

Lets jump into building a RPM with a real example. We will be packaging a php application.

Before we start I'm going to show you what the final spec file will look like and then I'll go on to explaining each part.

```

Name:           silex-app
Version:        1.0
Release:        1     
Group:          Development/Libraries
Summary:        PHP Silex App   
License:        Telstra Application Technologies
Source0:        %{name}-%{version}.zip
BuildRoot:      %{_tmppath}/%{name}-%{version}-buildroot

%description
This is an example of a PHP Silex app packaged as a RPM

%prep
# Unzip source file to BUILD dir
unzip %{SOURCE0}

%build
# The build section includes instructions, which are used to build and prepare the package for installation.

%install
# You must create these directories within the BuildRoot so you don't mess with the system directory
# Create the directories as it should look like in your system directory
# Normally in the make install process you set DESTDIR=${RPM_BUILD_ROOT} to achieve this

mkdir %{buildroot}
mkdir -p -m0755 %{buildroot}/var/
cp -r %{_builddir}/%{name}-%{version}/www %{buildroot}/var/
cp -r %{_builddir}/%{name}-%{version}/src %{buildroot}/var/
cp -r %{_builddir}/%{name}-%{version}/tests %{buildroot}/var/
cp -r %{_builddir}/%{name}-%{version}/composer.*  %{buildroot}/var/

%clean
rm -rf %{buildroot}

# Removed the unzipped file from the BUILD dir
rm -rf %{_builddir}/%{name}*

# Remove source file
rm %{SOURCE0}


%files

# Set the permissions to the files/directories created
%defattr(-,apache,apache,-)

# Speicfy file/dir created
# Note that exact files must be specified so that when you remove the rpm those files/dirs are also removed
/var/www
/var/src
/var/tests
/var/composer.*
```

##### Taking the .spec File Apart #####

**Preamble**

```
Name:           silex-app
Version:        1.4.5
Release:        1     
Group:          Development/Libraries
Summary:        PHP Silex App   
License:        Telstra Application Technologies
Source0:        %{name}-%{version}.zip
BuildRoot:      %{_tmppath}/%{name}-%{version}-buildroot
```

What is the `Name` field?
Name of the source file.

What is the `Group` field?
If you're publishing your RPM for public consumption then you should use existing Groups. If its only internal to your Organization you're free to call it whatever you want.

```
# Find out what groups you have
less /usr/share/doc/rpm-*/GROUPS
```

What is the `Source0` field?
As we've already covered your source file should be placed in the `SOURCES` dir and `Source0` should be name of the source file so it can be referred to when we build the RPM.

```
Name:           silex-app
Version:        1.4.5
# Source0 refers to silex-app-1.4.5.zip
Source0:        %{name}-%{version}.zip
```

You can also define multiple sources such as `Source1`, `Source2` etc.

**%prep**

```
%prep
# Unzip source file to BUILD dir
unzip %{SOURCE0}
```

Script command to  'prepare' (uncompress) the zip file to build it. Note that our `silex-app-1.4.5.zip` file will be unzipped to our `BUILD` dir when we build the RPM.

**%build**

Script commands to "build" the program (e.g. to compile it) and get it ready for installing. We're not doing that here.

**%install**

Here you are basically copying whatever you have in your `%{_builddir}` (BUILD dir) to `%{buildroot}` 

What is the `Buildroot`?
This was initially defined in the *preamble* section but it is best explained now.

As an example say you want to install your source files in `/opt/mypkgname/` and  you also want a config file in `/etc/mypkgname.conf`. To achieve this you would do the following in  the `%install` section:

```
%install
# Create the directory structure to mirror where the files are to be installed (when the RPM is installed) inside the Buildroot dir

# As per the definition in the preamble, Buildroot will be:
# /var/tmp/silex-app-1.4.5-buildroot

mkdir %{buildroot} 
mkdir -p %{buildroot}/opt/mypkg
mkdir -p %{buildroot}/etc


# Copy files across from the BUILD dir to Buildroot
cp -r %{_builddir}/%{name}-%{version}/mypkg %{buildroot}/opt/mypkg

cp %{_builddir}/%{name}-%{version}/mypkg.conf %{buildroot}/etc/mypkg.conf
```
**You are re-creating the tree that you want installed on your file system, with `%{buildroot}` as the equivalent of the target's root directory.**


Lets assume we have the following directory structure inside `silex-app` dir:

```
|-- config
|-- src 
|-- tests      
|-- www
|-- composer.json
|-- composer.lock    
```

When we build the RPM we want the `www`, `src`, `test` and the `composer` files copied to `/var` . Here's how you do it:

```
%install
mkdir %{buildroot}
mkdir -p -m0755 %{buildroot}/var/
cp -r %{_builddir}/%{name}-%{version}/www %{buildroot}/var/
cp -r %{_builddir}/%{name}-%{version}/src %{buildroot}/var/
cp -r %{_builddir}/%{name}-%{version}/tests %{buildroot}/var/
cp -r %{_builddir}/%{name}-%{version}/composer.*  %{buildroot}/var/
```

**%files**
Specifying all the files to be installed on the file system.

```
%files

# Set the permissions to the files/directories created
%defattr(-,apache,apache,-)

# Speicfy file/dir created
# Note that exact files must be specified so that when you remove the rpm those files/dirs are also removed
/var/www
/var/src
/var/tests
/var/composer.*
```

What is the `%defattr` attribute directive?

This allows setting of default attributes for files and directives.
```
# Files and Directories
%defattr(<file mode>, <user>, <group>, <dir mode>)

```

What if I want to give different permissions to different files?
```
#%attr(<mode>, <user>, <group>) file
%attr(0644, root, root) /var/www/.htaccess
```

All of the directories, sub-directories and files will be installed to the file system when the RPM is installed.


#### Building the RPM ####

Now that the .spec file is complete, we can proceed to building the RPM.

*Note:* Don't forget to place your source file in the `SOURCES` dir and set the build area in the `~/.rpmmacros` file!

Command to build the RPM:

`rpmbuild -ba rpmbuild/SPECS/myappname.spec`

If all goes well, your RPM will be available in the `rpmbuild/RPMS` dir.











