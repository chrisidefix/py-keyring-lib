=======================================
Installing and Using Python Keyring Lib
=======================================

.. contents:: **Table of Contents**

-------------------------
Installation Instructions
-------------------------
easy_install or pip
===================

Run easy_install or pip::

    $ easy_install keyring
    $ pip install keyring

Source installation
===================

Download the source tarball, and uncompress it, then run the install command::

    $ wget http://pypi.python.org/packages/source/k/keyring/keyring-0.1.tar.gz
    $ tar -xzvf keyring-0.1.tar.gz
    $ cd keyring-0.1
    $ python setup.py install


--------------------------
Configure your keyring lib
--------------------------

The python keyring lib contains implementations for several backends, including
**OSX Keychain**, **Gnome Keyring**, **KDE Kwallet** and etc. The lib will
automatically choose the keyring that is most suitable for your current
environment. You can also specify the keyring you like to be used in the config
file or by calling the ``set_keyring()`` function.

Customize your keyring by config file 
=====================================

This section is about how to change your option in the config file.

Config file path 
----------------

The configuration of the lib is stored in a file named "keyringrc.cfg". The file
can be stored in either of following two paths.

1. The working directory of the python
2. The home directory for current user

The lib will first look for the config file in the working directory. If no
config file exists **or** the config file is not write properly, the lib will
look up in the home folder.

Config file content 
-------------------

To specify a keyring backend, you need tell the lib the module name of the
backend, such as ``keyring.backend.OSXKeychain``. If the backend is not shipped 
with the lib, in another word, is made by you own, you need also tell the lib 
the path of your own backend module. The module name should be written after the 
**default-keyring** option, while the module path belongs the **keyring-path** 
option.

Here's a sample config file(The full demo can be accessed in the ``demo/keyring.py``):
::

    [backend]
    default-keyring=simplekeyring.SimpleKeyring
    keyring-path=/home/kang/pyworkspace/python-keyring-lib/demo/
    

Write your own keyring backend
==============================

The interface for the backend is defined by ``keyring.backend.KeyringBackend``. 
By extending this base class and implementing the three functions 
``supported()``,``get_password()`` and ``set_password()``, you can easily create 
your own backend for keyring lib.

The usage of the three functions:

* ``supported(self)`` : Return if this backend is supported in current 
  environment. The returned value can be **0**, **1** , or **-1**. **0** means 
  suitable; **1** means recommended and **-1** means this backend is not 
  available for current environment.
* ``get_password(self, service, username)`` : Return the stored password for the 
  ``username`` of the ``service``.
* ``set_password(self, service, username, password)`` : Store the ``password`` 
  for ``username`` of the ``service`` in the backend.

For an instance, there's the source code of the demo mentioned above. It's a 
simple keyring which stores the password directly in memory.

::

    """
    simplekeyring.py
    
    A simple keyring class for the keyring_demo.py
    
    Created by Kang Zhang on 2009-07-12
    """
    from keyring.backend import KeyringBackend
    
    class SimpleKeyring(KeyringBackend):
        """Simple Keyring is a keyring which can store only one
        password in memory.
        """
        def __init__(self):
            self.password = ''
    
        def supported(self): 
            return 0
    
        def get_password(self, service, username):
            return self.password
    
        def set_password(self, service, username, password):
            self.password = password
            return 0


Set the keyring in runtime
==========================

Besides setting the backend through the config file, you can also set the 
backend to use by calling the api ``set_keyring()``. The backend you passed in 
will be used to store the password in your application.

Here's a code snippet from the ``keyringdemo.py``. It shows the usage of 
``set_keyring()``
::

    # define a new keyring class which extends the KeyringBackend
    import keyring.backend
    class TestKeyring(keyring.backend.KeyringBackend):
        """A test keyring which always outputs same password
        """
        def supported(self): return 0
        def set_password(self, servicename, username, password): return 0 
        def get_password(self, servicename, username): 
            return "password from TestKeyring"
    
    # set the keyring for keyring lib
    import keyring
    keyring.set_keyring(TestKeyring())
    
    # invoke the keyring lib
    if keyring.set_password("demo-service", "tarek", "passexample") == 0:
        print "password stored successful"
    print "password", keyring.get_password("demo-service", "tarek")
    

-----------------------------------------------
Integrate the keyring lib with your application
-----------------------------------------------

API interface 
=============
The keyring lib has two functions: 

* ``get_password(service, username)`` : Returns the password stored in keyring. 
  If the password dose not exist, it will return None.
* ``set_password(service, username, password)`` : Store the password in the 
  keyring.

Example
=======
Here's an example of using keyring for application authorization. It can be 
found in the demo folder of the repository. Note that the faked auth function 
only returns true when the password equals to the username.
::
    
    """
    auth_demo.py
    
    Created by Kang Zhang 2009-08-14
    """
    
    import keyring
    import getpass
    import ConfigParser
    
    def auth(username, password):
        """A faked authorization function.
        """
        return username == password
    
    def main():
        """This scrip demos how to use keyring facilite the authorization. The 
        username is stored in a config named 'auth_demo.cfg'
        """
        # config file init
        config_file = 'auth_demo.cfg'
        config = ConfigParser.SafeConfigParser({
                    'username':'',
                    })
        config.read(config_file)
        if not config.has_section('auth_demo_login'):
            config.add_section('auth_demo_login')
    
        username = config.get('auth_demo_login','username')
        password = None
        if username != '':
            password = keyring.get_password('auth_demo_login', username)
    
        if password == None or not auth(username, password):
    
            while 1:
                username = raw_input("Username:\n")
                password = getpass.getpass("Password:\n")
    
                if auth(username, password):
                    break
                else:
                    print "Authorization failed."
            
            # store the username
            config.set('auth_demo_login', 'username', username)
            config.write(open(config_file, 'w'))
    
            # store the password
            keyring.set_password('auth_demo_login', username, password)
    
        # the stuff that needs authorization here
        print "Authorization successful."
    
    if __name__ == "__main__":
        main()
    
-------
Credits
-------

* Kang Zhang
* Tarek Ziadé