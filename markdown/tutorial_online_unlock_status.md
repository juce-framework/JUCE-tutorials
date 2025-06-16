# Tutorial: Unlock your plugins through online registration

Improve the security of your apps and plugins by locking their access until authorisation. Learn how to provide users with a mechanism to unlock your plugins through online registration of keys.

**LEVEL:** Advanced<br/>
**PLATFORMS:** Windows, macOS, Linux<br/>
**CLASSES:** [OnlineUnlockStatus](https://docs.juce.com/master/classOnlineUnlockStatus.html "A base class for online unlocking systems."), [OnlineUnlockForm](https://docs.juce.com/master/classOnlineUnlockForm.html "Acts as a GUI which asks the user for their details, and calls the appropriate methods on your Online..."), [KeyGeneration](https://docs.juce.com/master/classKeyGeneration.html "Contains static utilities for generating key-files that can be unlocked by the OnlineUnlockStatus cla..."), [RSAKey](https://docs.juce.com/master/classRSAKey.html "RSA public/private key-pair encryption class."), [TracktionMarketplaceStatus](https://docs.juce.com/master/classTracktionMarketplaceStatus.html "An implementation of the OnlineUnlockStatus class which talks to the Tracktion Marketplace server.")

> [!WARNING]
> This tutorial assumes elementary knowledge of RSA cryptography, certificates, network protocols and PHP syntax.

# Getting started

Download the demo project for this tutorial here: [PIP](https://docs.juce.com/tutorials/PIPs/OnlineUnlockStatusTutorial.zip) \| [ZIP](https://docs.juce.com/tutorials/ZIPs/OnlineUnlockStatusTutorial.zip) . Unzip the project and open the first header file in the Projucer.

If you need help with this step, see [Tutorial: Projucer Part 1: Getting started with the Projucer](/tutorials/tutorial_new_projucer_project/).

# The demo project

The demo project shows a very simple UI with two buttons, one of them is used to unlock access to the other one through registration. When the \"Unlock\" button is pressed, a registration form will open in order for the user to insert credentials. At the moment, the registration process will fail until we start implementing the back end server.

![](/_images/tutorial_online_unlock_status_screenshot1.png "The locked application window")

After successful implementation and authorisation of the application, you should see the following unlocked state of the app at the end of this tutorial.

![](/_images/tutorial_online_unlock_status_screenshot2.png "The unlocked application window")

# Introduction

As a plugin manufacturer, it can sometimes be frustrating to have your plugins cracked by hackers or even circulated free of charge without your consent. Although it can be difficult to make your applications full-proof against such threats, there are means to make this process more tedious for hackers. Sometimes you may want to offer a trial period for users to try your product before deciding on purchasing it or alternatively restrict certain features for free tier users.

Depending on the manufacturer, different methods are used to improve the security of plugins: some may use a third-party licensing solution such as PACE's iLok which requires a physical USB stick to carry the licenses or a proprietary system that authorises users by checking the credentials against a database on a remote web server.

In this tutorial, we look at the second method to register plugins via web server authorisation. The advantage of this method is that it can function both online as well as offline and does not require a physical device for registration. However since the plugin is registered to the machine for a period of time, this technique is more vulnerable to hacking compared to PACE's solution that requires authorisation every time the plugin launches.

So how does server authorisation work? If the machine is connected to the internet, we can perform what's called an online authorisation where the plugin sends the user credentials through a secure connection and probes the server for a license key. If the user has indeed been verified to have purchased the product, the server sends a key file back to the machine and the plugin can be unlocked. On the other hand if the machine is offline, the user can still download the key file from the web server via a different computer connected to the internet and the key file can then be copied and applied to the offline machine by loading it into the plugin.

## RSA encryption

Now let's talk a little bit about cryptography. How do we ensure that the information is not tampered with during transmission between the two parties? We use a widely-used secure encryption algorithm called RSA to ensure this. RSA works on the basis that a pair of keys is created using prime numbers with one of them being public and the other one being private. The public key can be shared publicly and it is used to encrypt the message. The private key is kept private by the receiving party and it is used to decrypt the message.

Due to the way RSA has been designed, only the private key owner can decrypt the message encrypted by the corresponding public key of the matching pair. This scenario can be described by the following diagram with Bob knowing the public key, Alice owning the private key and Eve a third-party trying to intercept the message:

![](/_images/dot_inline_dotgraph_8.svg "")

In the above case, Eve cannot read the message from Bob as she does not have the private key to decrypt the message. She can still send an encrypted message to Alice by using the public key but this is no use to Eve who wants to intercept Bob's message. This is usually the most common use case of RSA where some party wants to send a secret message but this is not the way RSA is used in the plugin registration process.

There is an alternative scenario to RSA used for plugin authentication where the private key is used to encrypt a message and the public key to decrypt it. This can be described by the following diagram with Bob being the machine on which the plugin is installed, Alice being the authenticating web server for registration and Eve a third-party trying to hack into the plugin:

![](/_images/dot_inline_dotgraph_9.svg "")

In this case, anyone with the public key can decrypt the message encrypted with the private key by Alice including Bob and Eve. However, if Eve tries to send an encrypted message to Bob without the corresponding private key from the matching pair, Bob will be able to check the authenticity of the message by decrypting it with the public key. He can then verify that the result is indeed valid as the data will contain garbage and therefore Eve cannot impersonate Alice.

Similarly to this, plugin authentication can be performed simply by checking whether the message sent by the server is valid provided that the server only sends the response if the credentials are correct against the database. Thankfully all this encryption and decryption procedure is handled for us by the JUCE library by classes such as [KeyGeneration](https://docs.juce.com/master/classKeyGeneration.html "Contains static utilities for generating key-files that can be unlocked by the OnlineUnlockStatus cla...") and [RSAKey](https://docs.juce.com/master/classRSAKey.html "RSA public/private key-pair encryption class.").

Now that we understand how RSA encryption works, let's start implementing the registration mechanism into our demo project.

# Implementing the marketplace status

In order to facilitate communication between the client using the application on a machine and a registration server that authorises the application, the JUCE library provides a base class that acts as an interface called the [OnlineUnlockStatus](https://docs.juce.com/master/classOnlineUnlockStatus.html "A base class for online unlocking systems.") class. This class provides a foundation to manage the registration status for the current machine on which the application is installed.

The first step towards implementing a store-specific marketplace is to inherit from the [OnlineUnlockStatus](https://docs.juce.com/master/classOnlineUnlockStatus.html "A base class for online unlocking systems.") class and an example of how to implement this subclass is described in the [TracktionMarketplaceStatus](https://docs.juce.com/master/classTracktionMarketplaceStatus.html "An implementation of the OnlineUnlockStatus class which talks to the Tracktion Marketplace server.") class. Since this implementation was designed specifically for the Tracktion marketplace, we need to create our own independent subclass but we will borrow the code that handles responses from the server as this can stay identical.

Let's first take a look at the implementation of the `TutorialMarketplaceStatus` class which derives from the [OnlineUnlockStatus](https://docs.juce.com/master/classOnlineUnlockStatus.html "A base class for online unlocking systems.") class. Notice that the `readReplyFromWebserver()` and `userCancelled()` functions were copied from the [TracktionMarketplaceStatus](https://docs.juce.com/master/classTracktionMarketplaceStatus.html "An implementation of the OnlineUnlockStatus class which talks to the Tracktion Marketplace server.") implementation for simplicity's sake.

Now the rest of the virtual functions from the base class need to be filled-in to complete our own marketplace implementation.

Insert a product ID for your application in the `getProductID()` function and make sure this ID matches the product ID on the server later on:

```cpp
juce::String getProductID() override
{
    return "TestApp";
}
```

The following `doesProductIDMatch()` function should verify whether the ID sent by the server matches the product ID inserted in the previous step:

```cpp
bool doesProductIDMatch (const juce::String& returnedIDFromServer) override
{
    return getProductID() == returnedIDFromServer;
}
```

Next, insert the name of your website in the `getWebsiteName()` function. This information will only be used to display the domain name to the user when contacting the server.

```cpp
juce::String getWebsiteName() override
{
    return "juce.com";
}
```

The next function is important as this is the address that will be contacted when attempting authorisation of the app and should point to the marketplace server. For the purpose of this tutorial, we point to the \"localhost\" or \"127.0.0.1\" IP address as the test server will run locally on our machine. The page it points to ends with a PHP extension as the server will use the PHP language for simplicity's sake.

```cpp
juce::URL getServerAuthenticationURL() override
{
    return juce::URL ("https://localhost:8443/auth.php");
}
```

> [!TIP]
>Notice that the protocol used to communicate to the server has to be HTTPS as the authentication process only works over a secure connection. The 8443 port reflects this choice and denotes the default SSL port.

When the registration process starts, the client app will request authentication with the credentials supplied by the user and the server will respond with an encrypted message. This message containing the key file to unlock the app can only be decrypted using the public RSA key that corresponds to the private RSA key used to encrypt the message on the server side.

The `getPublicKey()` function must return this public key and will be filled later on in the key generation step of this tutorial. Prepare the function as follows and leave the field blank for the moment:

```cpp
juce::RSAKey getPublicKey() override
{
    return juce::RSAKey ("INSERT_PUBLIC_KEY_HERE");
}
```

The `TutorialMarketplaceStatus` class now has all the necessary information to communicate with the marketplace server and retrieve the keys.

# Provide a registration form

There are several ways to display a registration form for the user to fill in an app or plugin. We can either overlay a regular [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") on top of our interface or alternatively pop up a [DialogWindow](https://docs.juce.com/master/classDialogWindow.html "A dialog-box style window.") on top of our main window. The latter option is highly discouraged when developing plugins as this can freeze a simple scanning routine when validating plugins and it can also cause problems if a DAW shuts the plugin down while the registration window is still active.

Therefore in this section, we opt for the former option to ensure compatibility with both apps and plugins. In order to simplify the process of implementing a registration user interface, the JUCE library offers the [OnlineUnlockForm](https://docs.juce.com/master/classOnlineUnlockForm.html "Acts as a GUI which asks the user for their details, and calls the appropriate methods on your Online...") class that displays two fields for an email and a password and provides seamless implementation for registration in conjunction with the [OnlineUnlockStatus](https://docs.juce.com/master/classOnlineUnlockStatus.html "A base class for online unlocking systems.") class.

Create a class named `TutorialUnlockForm` and derive from [OnlineUnlockForm](https://docs.juce.com/master/classOnlineUnlockForm.html "Acts as a GUI which asks the user for their details, and calls the appropriate methods on your Online...") as follows:

```cpp
class TutorialUnlockForm : public juce::OnlineUnlockForm
{
public:
    TutorialUnlockForm (TutorialMarketplaceStatus& status)
        : OnlineUnlockForm (status, "Please provide your email and password.")
    {
    }
```

Here we also pass a reference to our `TutorialMarketplaceStatus` class implemented in the previous section and provide a descriptive message to instruct the user in the class constructor.

In the `MainContentComponent` , declare three private member variables repectively an instance of a `TutorialMarketplaceStatus` and a `TutorialUnlockForm` object as well as a boolean to store whether the application was unlocked.

```cpp
TutorialMarketplaceStatus marketplaceStatus;
TutorialUnlockForm unlockForm;

bool isUnlocked = false;

JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR (MainContentComponent)
};
```

Modify the constructor to initialise the `TutorialUnlockForm` by passing the `TutorialMarketplaceStatus` object and add the form as a child [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") as shown here:

```cpp
class MainContentComponent : public juce::Component,
                             private juce::Timer
{
public:
    //==============================================================================
    MainContentComponent()
        : unlockForm (marketplaceStatus)
    {
```

```cpp
addChildComponent (unlockForm);

setSize (400, 250);
startTimer (100);
}
```

Notice here that we also inherit from the [Timer](https://docs.juce.com/master/classTimer.html "Makes repeated callbacks to a virtual method at a specified time interval.") class and call the `startTimer()` function which will be used later on to check whether the app is unlocked.

In the `resized()` method, set the size of the form to cover up most of the interface by calling the `centreWithSize()` function like so:

```cpp
void resized() override
{
    unlockForm.centreWithSize (300, 200);
```

The `showForm()` helper function will be called when the \"Unlock\" button is pressed by the user. Since the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects.") was simply added with the `addChildComponent()` function instead of `addAndMakeVisible()` , the form is hidden by default thus simply call `setVisible()` here to show the form:

```cpp
void showForm()
{
    unlockForm.setVisible (true);
}
```

When the registration process is cancelled or succeeds, the form needs to be dismissed and the [OnlineUnlockForm](https://docs.juce.com/master/classOnlineUnlockForm.html "Acts as a GUI which asks the user for their details, and calls the appropriate methods on your Online...") class automatically calls the `dismiss()` callback to do this. In the `TutorialUnlockForm` declaration, override this function by simply calling `setVisible()` to hide the [Component](https://docs.juce.com/master/classComponent.html "The base class for all JUCE user-interface objects."):

```cpp
void dismiss() override
{
    setVisible (false);
}
};
```

As seen previously in the constructor of the `MainContentComponent` , a timer is launched to periodically check whether the application was successfully unlocked in the `timerCallback()` function:

```cpp
void timerCallback() override
{
    if (!isUnlocked && marketplaceStatus.isUnlocked())
    {
        isUnlocked = true;
        unlockApp();
    }
}
```

Here we check whether the app is still locked with the `isUnlocked` local variable and if so, ask the [OnlineUnlockStatus](https://docs.juce.com/master/classOnlineUnlockStatus.html "A base class for online unlocking systems.") instance whether the app is still locked via the `isUnlocked()` function. If the app has been authorised, we switch the local variable to prevent subsequent iterations and call the following `unlockApp()` helper function:

```cpp
void unlockApp()
{
    secretButton.setEnabled (true);
    unlockButton.setEnabled (false);

    unlockLabel.setText ("Status: Unlocked", juce::dontSendNotification);
    unlockLabel.setColour (juce::Label::textColourId, juce::Colours::green);
}
```

This function simply enables the button to access the secret feature and disables the button to unlock the app while showing the new authorisation status.

Finally the `checkFeature()` function is called when the user clicks on the secret feature button and simulates the behaviour of the unlocked application:

```cpp
void checkFeature()
{
    if (marketplaceStatus.isUnlocked())
        DBG ("App unlocked!");
    else
        DBG ("Beware of hackers!");
}
```

This function is important as it provides another layer of security to our application. Notice here that the function checks again with the [OnlineUnlockStatus](https://docs.juce.com/master/classOnlineUnlockStatus.html "A base class for online unlocking systems.") instance whether the app is still unlocked. The reason behind this is because not only do hackers need to crack this function to unlock the app, they will need to replace all instances of this function in your application code in order to fully breach the restricted features of the app.

> [!TIP]
>It is therefore recommended to check for the authorisation status of your application in as many places as possible in the code to provide a more robust security mechanism.

We can now display the registration form if we run the application.

![](/_images/tutorial_online_unlock_status_screenshot3.png "The registration form")

However, if we try to register the app with an arbitrary email and password combination, we will see the following error message:

![](/_images/tutorial_online_unlock_status_screenshot4.png "Connection issue dialog")

This is because the application cannot connect to the authentication server as the setup is covered in the next section of this tutorial.

> [!TIP]
>The source code for this modified version of the code can be found in the `OnlineUnlockStatusTutorial_02.h` file of the demo project.

# Setting up the back end server

> [!TIP]
>This section covers the setup instructions for a local test server on macOS and Windows using PHP but feel free to use any type of back end server whether it be Java, Ruby or Python etc...

Since the [OnlineUnlockStatus](https://docs.juce.com/master/classOnlineUnlockStatus.html "A base class for online unlocking systems.") class handles replies from the server in the XML format, we have to supply an XML response using PHP. The application will send the credentials to the [URL](https://docs.juce.com/master/classURL.html "Represents a URL and has a bunch of useful functions to manipulate it.") supplied in the `getServerAuthenticationURL()` function of our `TutorialMarketplaceStatus` implementation which was defined in the first section of the tutorial as \"https://localhost:8443/auth.php\".

Hence, let's have a look at the `auth.php` file supplied with the demo project in the `Resources` folder and examine how the response was crafted in PHP:

```php
<?php
function sendResponse() // [1]
{
    $response = '<?xml version="1.0" encoding="utf-8"?>'; // [2]
    if ($_POST['email'] === 'test@example.com' and $_POST['pw'] === 'test') // [3] {
        $response .=
            '<MESSAGE message="Thanks for registering our product!"><KEY>INSERT_KEY_HERE</KEY></MESSAGE>';
    } else {
        $response .=
            '<ERROR error="Sorry, we were not able to authorise your request. Please provide a valid email address and password."></ERROR>';
    }

    return $response;
}

header('Content-type: text/xml; charset=utf-8'); // [4]
echo sendResponse();
?>

```

- [1]: We first define a function called `sendResponse()` that will be called later on to display the XML content.
- [2]: Then we prepare an XML document by assigning the metadata header into a local PHP variable.
- [3]: Next, we check whether the credentials correspond to the test user by comparing with test email and password strings received via POST. If the credentials are matching, we append the success message along with an encrypted key file that we will insert in the next section. Otherwise, we append an error message explaining the reason why the authorisation failed.
- [4]: Finally, we set the response header to indicate that the content is in the XML format and send the response with the `echo` function.

At this point, if we decide to run the server locally and try to contact the authorisation server from the app, we would receive a security warning telling us that the connection is not private or secure. For the [OnlineUnlockStatus](https://docs.juce.com/master/classOnlineUnlockStatus.html "A base class for online unlocking systems.") instance to send the credentials securely, we need to create a secure connection using SSL via the HTTPS protocol.

To do this locally for testing purposes, we can create a self-signed certificate and use tunneling to access the underlying unsecure HTTP protocol through HTTPS.

> [!WARNING]
> Make sure that you have OpenSSL, PHP and \"stunnel\" installed on your computer to test the back end server. OpenSSL will be necessary to generate the self-signed certificate, PHP offers a built-in web server that you can run locally on your machine and \"stunnel\" allows you to tunnel a secure connection into it.

Navigate to the `Resources` folder of the project and create a self-signed certificate by running the following command from the terminal:

```bash
// Creates an RSA key and certificate pair.

// macOS
openssl req -new -newkey rsa:4096 -x509 -nodes -sha256 -days 365 -keyout stunnel.key -out stunnel.cert

// Windows
C:\\path-to-openssl\\openssl req -new -newkey rsa:4096 -x509 -nodes -sha256 -days 365 -keyout stunnel.key -out stunnel.cert
```

When prompted to provide information for the certificate, skip all the fields except for the common name or CN entry where you should enter \"localhost\".

> [!TIP]
>Make sure that the domain name is not misspelt as this would cause problems when connecting to the server.

This will create two files forming a pair of RSA 4096-bit key and certificate using the SHA-256 hash algorithm and an expiry date of one year. The X.509 standard denotes the self-signed policy we are interested in. To simplify things, we combine the generated key and certificate files into one PEM file like this:

```bash
// Combines the key and certificate into a PEM file.

// macOS
cat stunnel.key stunnel.cert > stunnel.pem

// Windows
type stunnel.key stunnel.cert > stunnel.pem
```

On macOS, double-click the PEM file in the `Finder` to add the certificate to your keychain. This will open the `Keychain Access` application located in the `Applications/Utilities` folder. Double-click on the newly-added certificate in the list to open the detailed view and expand the **Trust** view. Select the \"Always Trust\" option to allow the certificate to be trusted by the current user account of your mac as shown below:

![](/_images/tutorial_online_unlock_status_screenshot5.png "The self-signed certificate")

On Windows, open the `Microsoft Management Console` by typing \"MMC\" in the `Run` prompt and navigate to **File \> Add Snap-in...** and select \"Certificates\". Click on **Add** to add certificates for the current user and click **OK** to finish.

![](/_images/tutorial_online_unlock_status_screenshot9.png "Adding the certificate snap-in")

Then import the self-signed certificate into **Trusted Root Certification Authorities \> Certificates** by navigating to the corresponding folder in the left panel and selecting **Action \> All Tasks \> Import...** from the menu items. You will be guided through the steps to locate the certificate at which point you should select the file with the `.cert` extension as this is the format accepted by the MMC.

![](/_images/tutorial_online_unlock_status_screenshot10.png "Importing the certificate")

> [!WARNING]
> Do not use a self-signed certificate or tunneling for real applications as this is not considered as secure as a CA-signed certificate. This method is only used for demonstrative purposes.

Next, let's configure the tunneling protocol by opening the `stunnel.conf` file supplied with the demo project in the `Resources` folder:

```
[https]
accept = 8443
connect = 8080
cert = stunnel.pem
TIMEOUTclose = 0
```

Here we define the incoming and outgoing ports to connect to and from as well as the certificate file created in the previous step to secure the link. The application will attempt to connect to port 8443 and the tunneling will link to port 8080 where the PHP server will be running.

We can finally launch the server by navigating to the `Resources` directory and running the following command from the terminal:

```bash
// Starts the built in PHP web server and tunnels a secure connection.

// macOS
php -S localhost:8080 & stunnel stunnel.conf

// Windows
C:\\path-to-php\\php -S 127.0.0.1:8080 & C:\\path-to-stunnel\\stunnel stunnel.conf
```

This will launch the local server to port 8080 and tunnel the connection to port 8443 for the application to access securely. Open the authentication [URL](https://docs.juce.com/master/classURL.html "Represents a URL and has a bunch of useful functions to manipulate it.") \"https://localhost:8443/auth.php\" in your favourite browser to check that the server is running. You should see the error message declared in the `auth.php` file in the form of an XML response.

If all works properly, launch the application and attempt authorisation by submitting an arbitrary email and password combination. The error message you receive should be different from the previous attempt and should match the message shown in the browser like so:

![](/_images/tutorial_online_unlock_status_screenshot6.png "Error message dialog")

This means that the application was successful in contacting the server and handling the response however we still have to generate and send the correct key file to unlock the application.

Stop the server by hitting \"Ctrl+C\" on the keyboard or run the following commands to stop the server and the tunneling:

```bash
// Stops any running PHP server processes and the HTTPS to HTTP tunneling.

// macOS
killall php & killall stunnel

// Windows
taskkill /f /im php.exe & taskkill /f /im stunnel.exe
```

# Generating security keys

Let's have a look at generating security keys as a final step.

You will find a helper Console application called `KeyGenerator` in the `Resources` folder of the project that will generate an RSA key pair to use for authenticating this application. You only have to generate keys once for every app or plugin as these can remain the same for the lifetime of the project (unless the private key is somehow compromised in which case a new pair has to be created).

If we take a look at the `main()` function of the `KeyGenerator` app, we can see that all it does is call the `createKeyPair()` function from the [RSAKey](https://docs.juce.com/master/classRSAKey.html "RSA public/private key-pair encryption class.") class with a key size of 256 bits and prints the result on screen:

```cpp
juce::RSAKey publicKey;
juce::RSAKey privateKey;

juce::RSAKey::createKeyPair (publicKey, privateKey, 256);

std::cout << "Public Key: " << publicKey.toString() << std::endl
          << "Private Key: " << privateKey.toString() << std::endl
          << std::endl;
```

Build the `KeyGenerator` app and run the following command to retrieve the keys:

```bash
// Run the program and copy the keys.

// macOS
./KeyGenerator

// Windows
KeyGenerator.exe
```

Copy and paste the public key from the previous step into the `getPublicKey()` function of the `TutorialMarketplaceStatus` class:

```cpp
return juce::RSAKey ("INSERT_PUBLIC_KEY_HERE");
```

This will allow the application to decrypt the message from the server provided the message is authentic and was encrypted using the matching private key.

In order for the server to generate an authentic key file when requested from the app, it must own the private key generated in the previous step to encrypt the message. In a real production scenario, the key generation step must be done on the server programmatically but for the purpose of this tutorial, we will perform this step manually and copy paste the key file into the PHP script.

Navigate to the `Resources` folder once again to find another helper Console application called `Unlocker` . This will create the necessary key file to unlock the app by calling the `generateKeyFile()` function from the [KeyGeneration](https://docs.juce.com/master/classKeyGeneration.html "Contains static utilities for generating key-files that can be unlocked by the OnlineUnlockStatus cla...") class:

```cpp
// args[0]: app-name
// args[1]: user-email
// args[2]: username
// args[3]: machine-numbers
// args[4]: private-key

std::cout << juce::KeyGeneration::generateKeyFile (args[0], args[1], args[2], args[3], juce::RSAKey (args[4])) << std::endl;
```

The function takes as argument the app name, user email and username, the machine IDs authorised for use as well as the RSA private key to encrypt the message. You can find your corresponding machine ID when running the demo project by checking the debugger log when attempting registration.

Build the `Unlocker` app and run the following command to retrieve the encrypted key file:

```bash
// Run the program and copy the key.

// macOS
./Unlocker TestApp test@example.com testuser INSERT_MACHINE_ID_HERE INSERT_PRIVATE_KEY_HERE

// Windows
Unlocker.exe TestApp test@example.com testuser INSERT_MACHINE_ID_HERE INSERT_PRIVATE_KEY_HERE
```

Copy and paste the key file that starts with the \"#\" character from the previous step into the `sendResponse()` function of the PHP script and include the \"#\" character while doing so:

```php
$response .= '<MESSAGE message="Thanks for registering our product!"><KEY>INSERT_KEY_HERE</KEY></MESSAGE>';
```

Launch the server then run the application to attempt authorisation by submitting the appropriate email and password combination as defined in the PHP script:

![](/_images/tutorial_online_unlock_status_screenshot7.png "Filling in the credentials")

If all goes well, you should see the following success message and the application should unlock after the registration form dismisses itself:

![](/_images/tutorial_online_unlock_status_screenshot8.png "Registration success dialog")

# Summary

In this tutorial, we have learnt how to unlock plugins through online registration. In particular, we have:

- Understood the principles of cryptography with RSA.
- Provided a UI for the user to unlock the app through a form.
- Generated security keys for the unlocking mechanism of the app.
- Built a simple secure local server to act as a back end.
- Unlocked features of the application depending on the registration status.

# See also

- [Tutorial: Package your app or plugin for distribution](/tutorials/tutorial_app_plugin_packaging/)
- [Tutorial: App analytics collection](/tutorials/tutorial_analytics_collection/)
- [Tutorial: In-App Purchases on desktop and mobile devices](/tutorials/tutorial_in_app_purchases/)
- [Tutorial: Push Notifications on desktop and mobile devices](/tutorials/tutorial_push_notifications/)
