# Publishing NW.js apps on the Mac App Store


# Getting the MAS-compatible executable

on @alexeyst

# Packaging your app

## Standard packaging

https://github.com/nwjs/nw.js/wiki/How-to-package-and-distribute-your-apps

## Additional steps

Remove ffmpeg library

Remove .DS_Store files

Remove crash_inspector

# Installing your app icon

convert png > icns

icon path

format

# Configuring plist file

__name__
__bundle_identifier__
__version__
__bundle_version__
__copyright__
__app_category__    public.app-category.utilities
__app_sec_category__    public.app-category.productivity

https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/LaunchServicesKeys.html

# Configuring your machine with your developer account

create certificate
ask certificate
install
get identity

# Signing the app

Sign 3 helpers and app

Test

# Sending the app

Package with productbuild

Send on application loader

# Submitting your app

Choose binary, fill fields, send


# Thanks

[Alexey Stoletny](https://github.com/alexeyst)
[Trevor Linton](https://github.com/trevorlinton)
[Roger Wang](https://github.com/rogerwang)
