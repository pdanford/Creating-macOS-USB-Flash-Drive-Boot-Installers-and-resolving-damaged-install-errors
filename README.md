Creating macOS USB Flash Drive Boot Installers (and resolving "damaged" install errors)
=======================================================================================

This info details how to create bootable USB flash drive installers for macOS along with dealing with failed OS installs due to "damaged" app bundles.

pdanford - July 2020

---
1. Format a USB 8GB (16GB for Catalina) flash drive as Mac OS Extended (Journaled) named **UNTITLED**.
2. cd to the directory where your macOS installer app is located.
3. Use one of the commands below to create the desired bootable USB flash drive installer.

Mavericks (10.9) through Sierra (10.12), use

    sudo "./<installer_name.app>/Contents/Resources/createinstallmedia" --volume /Volumes/UNTITLED --applicationpath "./<installer_name.app>" --nointeraction

Where &lt;installer_name.app&gt; is the installer app bundle you're using (e.g. "Install OS X Mavericks.app").

For High Sierra (10.13) and above, use

    sudo "./<installer_name.app>/Contents/Resources/createinstallmedia" --volume /Volumes/UNTITLED --nointeraction

OS versions before Mavericks (e.g. Mountain Lion) have an older procedure. See [here](https://appducate.com/2012/12/install-os-x-mountain-lion-from-usb-flash-drive).

---
#### About booting from USB flash drives

Note that newer macbooks need to enable booting from external drives in the T2 security chip options from the macOS Utilities GUI.

1) Get to the macOS Utilities GUI menu by rebooting while holding <kbd>command</kbd><kbd>R</kbd>.
2) Select **Utilities->Startup Security Utility**.
3) For **Secure Boot**, select No Security and for **Allowed Boot Media**, select Allow booting from external or removable media...

Rebooting while holding <kbd>option</kbd> will let you select the USB flash drive as the boot drive.

---
#### To get around errors like “This copy of the Install macOS Mojave.app application is damaged, and can’t be used to install macOS.”

When using a macOS installer app downloaded from the App Store and saved for later use to do a fresh install from USB flash drive or install as a virtual machine in e.g. VMware, the above error may be displayed. The bundle is not actually damaged and the work-around is fairly straight forward; read on.

This happens because the security certificate embedded in the installer bundle expired before the install date. To work around:

1. If you want to see specifics of the expired certificate(s), you can investigate the certs of the macOS Installer with cli tools which can show the expired certificate that caused the error message:
    1. For an installer app itself, use codesign to extract the certificates to the current directory:<br>
       **codesign --display --extract-certificates Install\ macOS\ Mojave.app**<br>
    2. Then you can view valid dates in them with:<br>
       **for C in $(ls codesign\*); do openssl x509 -inform DER -in "${C}" -text | grep -i -A 2 "valid"; done**<br>
      or use Quick Look:<br>
       **for C in $(ls codesign\*); do qlmanage -c public.x509-certificate -p "${C}"; done**
    3. For combo updates, use pkgutil:<br>
       **pkgutil --check-signature macOSUpdCombo10.14.6.pkg**
2. When installing from a bootable USB flash drive with a macOS installer or doing drag-and-drop of the installer app onto VMware **File->New... 'Install from disc or image'** window, the macOS Utilities GUI will eventually come up.
3. When the GUI is displayed, disconnect from wifi/network and go to **Utilities->Terminal** to set the system date in the range of the expired installer cert (e.g. if expired on October 24, 2019, execute **date 030300002019**) and quit terminal with <kbd>command</kbd><kbd>Q</kbd>.
4. Select **Install macOS** from the macOS Utilities menu list. Do not connect back to wifi/network until the OS user setup phase has started or else the install process will get the current date from the net and fail.

---
For additional info, see:<br>
[How to create a bootable installer for macOS](https://support.apple.com/en-us/HT201372) <br>
[Beware apple security certificates after 24 October](https://eclecticlight.co/2019/10/18/beware-apple-security-certificates-after-24-october-they-may-have-expired) <br>
[Get the certificate expiration date of an application](https://stackoverflow.com/questions/25391429/get-the-certificate-expiration-date-of-an-application-using-codesign-on-mac) <br>
[Fix macOS install application damaged - can't be used error](https://osxdaily.com/2019/10/24/fix-install-macos-application-damaged-cant-be-used-error-mac) <br>

---
:scroll: [MIT License](README.license)
