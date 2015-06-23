# Publishing NW.js apps on the Mac App Store: a detailed guide

1. [Important informations](#important-informations)
2. [Packaging your app](#packaging-your-app)
3. [Installing your app icon](#installing-your-app-icon)
4. [Configuring the plist file](#configuring-the-plist-file)
5. [Signing the app](#signing-the-app)
6. [Uploading the app](#uploading-the-app)
7. [Submitting the app for validation](#submitting-the-app-for-validation)
8. [Thanks](#thanks)

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

## Signing the app

*Prior to signing your app, you have to request and install certificates from the Apple Member Center. To do so, you can follow [this guide](CERTIFICATES.md).*

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

We assume that you have saved both files as `child.plist` and `parent.plist`.

You can now run the signing commands:

```bash

export IDENTITY=LK12345678 # Update with your identity (saved when generating the certificates)
export BUNDLE_ID=com.yourcompanyname.yourappname
export PARENT_PLIST=/path/to/parent.plist
export CHILD_PLIST=/path/to/child.plist
export APP_PATH=/path/to/yourapp.app

codesign --deep -s IDENTITY -i BUNDLE_ID --entitlements CHILD_PLIST $APP_PATH"/Contents/Frameworks/nwjs Helper.app"

codesign --deep -s IDENTITY -i BUNDLE_ID --entitlements CHILD_PLIST $APP_PATH"/Contents/Frameworks/nwjs Helper EH.app"

codesign --deep -s IDENTITY -i BUNDLE_ID --entitlements CHILD_PLIST $APP_PATH"/Contents/Frameworks/nwjs Helper NP.app"

codesign --deep -s IDENTITY -i BUNDLE_ID --entitlements PARENT_PLIST APP_PATH

```


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
