# Publishing NW.js apps on the Mac App Store: a detailed guide

1. [Important informations](#important-informations)
1. [Packaging your app](#packaging-your-app)
2. [Installing your app icon](#installing-your-app-icon)
3. [Configuring the plist file](#configuring-the-plist-file)
3. [Requesting certificates to sign your app](#requesting-certificates-to-sign-your-app)
4. [Signing the app](#signing-the-app)
5. [Uploading the app](#uploading-the-app)
6. [Submitting the app for validation](#submitting-the-app-for-validation)
7. [Thanks](#thanks)

## Important informations

Please be aware that **NW.js** currently does not officially supports publishing on the Mac App Store.

NW.js is based on Chromium, which uses *private APIs* ([what is a private API](http://stackoverflow.com/questions/3000681/what-are-private-apis)), that are not allowed on the MAS.

So to comply with the MAS rules, you will need to use a patched, custom binary in which those APIs are no longer used.

This binary has been tested and is used to run several applications on the MAS, but keep in mind that you may come across unexpected issues.

If you want more informations, or compile the binary by yourself, you can check [this document](COMPILING.md).

## Packaging your app

The custom `nwjs.app` application you need to use can be downloaded on the alexeyst's [node-webkit-macappstore](https://github.com/alexeyst/node-webkit-macappstore) repository.

You have to get the `nwjs.app` file (or `node-webkit.app`, depending on the version you want to use).

### Standard packaging

Package your app by following the regular procedure (if you don't know how to do it, check the [official documentation page](https://github.com/nwjs/nw.js/wiki/How-to-package-and-distribute-your-apps)).

Roughly, you have to:

* Rename your app root directory (the one containing the `package.json` file) to `app.nw`
* Move this directory to `nwjs.app/Contents/Resources`
* Rename the app to what you want - let's assume it is `yourapp.app`

### Additional steps

To ensure proper validation, you have to check thoses steps:

Delete the FFMPEG library:

```bash
rm yourapp.app/Contents/Frameworks/nwjs Framework.framework/Libraries/ffmpegsumo.so
```

Delete `.DS_Store` files that may have been generated when you were working on your app:

```bash
cd yourapp.app && find . -name "*.DS_Store" -type f -delete
```

Remove the `crash_inspector` file, if it exists in your app:

```bash
rm yourapp.app/Contents/Frameworks/crash_inspector
```

## Installing your app icon

You will need a `.icns` file, size `1024x1024`.

If you work with a `.png` file, you can convert it with a tool like [png2icns](https://github.com/daveish/png2icns).

Once your icon is ready, move it to `yourapp.app/Contents/Resources/nw.icns` , by overriding the existing one.

## Configuring the plist file

The `Info.plist` file (located in `yourapp.app/Contents`) is used on OS X to display your app informations, and on the Mac App Store for identification purposes. You will need to update it.

You may start from this [sample file](Info.plist).

You can override the existing file, and update the following fields:

| Field | Informations
| --- | --- |
| `__name__` | Readable app name |
| `__bundle_identifier__` | App identifier, looks like `com.yourcompanyname.yourappname` |
| `__version__` | Readable app version, looks like `1.4.8` |
| `__bundle_version__` | A unique, numeric app version - you can use the version as a base, like `148` for `1.4.8` (this solution is more readable than starting from `1` and counting) (this is used on the MAS only) |
| `__copyright__` | A readable copyright message, appears when doing a `CMD-I` on your app |
| `__app_category__` | The app category, looks like `public.app-category.entertainment` ([complete list](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/LaunchServicesKeys.html)) |
| `__app_sec_category__` | An additional category (same format as above) |

The sample file contains the minimum, required fields that you need on the MAS.

You may need to add other fields, to associate a file extension to your app, for instance.

## Requesting certificates to sign your app

*This step assumes you have an active developer account. If this is not the case, you can visit the [Apple Developer Program](https://developer.apple.com/programs/) page, and click on Enroll. Applying costs $100 per year.*

Apps on the Mac App Store are signed. To sign your app, you will need to have certificates installed on your machine.

A few words about code signing:

> Code signing is a security technology, used in OS X, that allows you to certify that an app was created by you.
> Once an app is signed, the system can detect any change to the app—whether the change is introduced accidentally or by malicious code.

[Source](https://developer.apple.com/library/mac/documentation/Security/Conceptual/CodeSigningGuide/Introduction/Introduction.html)

To sign your app, two certificates are needed:

* A **Mac App Distribution** certificate - used to sign the `.app` file
* A **Mac Installer Distribution** certificate - used to sign the `.pkg` file that will be uploaded to the MAS

### Preparing the request

To request a certificate you need a `.certSigningRequest` file.

Open `/Applications/Utilities/Keychain Access.app`; in the app menu, and select *Request a Certificate From A Certificate Authority...*:

![Capture](screenshots/keychain_request_menu.jpg)

Fill the needed informations in the assistant:

![Capture](screenshots/keychain_request_assistant.jpg)

1. Your email address
2. The name of the certificate
3. The *CA Email* field tells it is required, but you have to leave it blank

Save the file on the disk. Its name is `CertificateSigningRequest.certSigningRequest`.

### Generating the certificates from the Apple Member Center

Login to the [Member Center](https://developer.apple.com/membercenter), and go to the *Certificates, Identifiers & Profiles* page, which looks like this:

![Capture](screenshots/certificates_splashscreen.jpg)

In the *Mac Apps > Certificates* page, click on the **+** button to add a new *Mac App Store certificate*:

![Capture](screenshots/certificate_type.jpg)

Then, select *Mac App Distribution*:

![Capture](screenshots/certificate_generate.jpg)

Upload your `CertificateSigningRequest.certSigningRequest` file, when asked:

![Capture](screenshots/import_csr.jpg)

Finally, download the generated certificate on your machine:

![Capture](screenshots/certificate_ready.jpg)

Then, you have to renew those steps, by selecting *Mac Installer Distribution* instead of *Mac App Distribution*, on the concerned step.

### Installing the certificates

When both certificates have been downloaded, you have to import them in **Keychain Access**, by double-clicking on them.

The installed certificates look like this:

![Capture](screenshots/installed_certificates.jpg)

Note the string in parenthesis (beginning with `LK7` on the capture).

In the signin step, we will refer to it as the **identity**.

## Signing the app

### Configuring the permissions

Due to the `NW.js` structure, we have to sign the four following apps:

* `yourapp.app/Contents/Frameworks/nwjs Helper.app`
* `yourapp.app/Contents/Frameworks/nwjs Helper EH.app`
* `yourapp.app/Contents/Frameworks/nwjs Helper NP.app`
* `yourapp.app`

To do so, we use two entitlement files ([more informations](https://developer.apple.com/library/mac/documentation/Miscellaneous/Reference/EntitlementKeyReference/Chapters/AboutEntitlements.html)).

The parent entitlements will be applied on the main app, and lists what the app is allowed to do.

The child entitlements will be applied on the three helpers, and basically say that they inherit from the parent.

The child file looks like this:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>com.apple.security.app-sandbox</key>
	<true/>
	<key>com.apple.security.inherit</key>
	<true/>
</dict>
</plist>
```

The parent file looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>com.apple.security.app-sandbox</key>
	<true/>
</dict>
</plist>
```

You have to add rules to suit your app needs. For instance, if you want your app to connect to the internet, you will have to add:

```xml
<key>com.apple.security.network.client</key>
<true/>
```

Or, if you want to read and write files, once they have been selected by the user (through a file dialog, for instance):

```xml
<key>com.apple.security.files.user-selected.read-write</key>
<true/>
```

Complete list of rules is available [here](https://developer.apple.com/library/ios/documentation/Miscellaneous/Reference/EntitlementKeyReference/Chapters/EnablingAppSandbox.html).

### Signing commands

We assume that you have saved both files, `child.plist` and `parent.plist`, in the same directory than `yourapp.app`.

You can now start the signing commands:

@todo

## Uploading the app

@todo

Package with productbuild

Send on application loader

## Submitting the app for validation

@todo

Choose binary, fill fields, send

## Thanks

[Alexey Stoletny](https://github.com/alexeyst)
[Trevor Linton](https://github.com/trevorlinton)
[Roger Wang](https://github.com/rogerwang)
