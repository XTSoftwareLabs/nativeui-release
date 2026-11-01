# Getting started

## Install

Extract the release ZIP into a folder, for example `C:\Tools\nativeboat`, and
add the folder to your user `PATH`. Then check it in a new terminal:

```powershell
nativeboat --version
```

## Prepare your project

Install the project's dependencies as you would for an Electron build:

```powershell
cd C:\src\my-app
npm install          # or pnpm install, yarn
```

## Check your project

```powershell
nativeboat analyze
```

Reports which Electron, Node.js and web features the project uses and how
well each is supported, with where each is used. Add `--json` for a JSON
report. [Compatibility](compatibility.md) lists the features and their support levels.

## Build

```powershell
nativeboat build
```

Writes `<productName>.exe` into the project's output folder (`dist` unless
the project's electron-builder configuration names another). Run it like any
Windows program.

| Option | Meaning |
| --- | --- |
| `--out <DIR>` | the output folder |
| `--sign <COMMAND>` | signs the executable; see [Signing](#signing) |
| `--installer <DIR>` | where the installer and its update metadata go; see [Installers and updates](#installers-and-updates) |
| `--work-dir <DIR>` | where build copies of projects are kept |

Both commands take the project folder as an argument and default to the
current folder:

```powershell
nativeboat build C:\src\my-app --out C:\builds\my-app
```

If the project's packaging script builds something before packaging, such as
a helper program, `nativeboat build` runs those steps too, so install the
tools they need.

### Signing

`{file}` in the command stands for the executable:

```powershell
nativeboat build --sign "signtool sign /a /fd SHA256 /tr http://timestamp.digicert.com /td SHA256 {file}"
```

## Debug

Start the application from a terminal to see its console output and errors:

```powershell
Start-Process dist\MyApp.exe -NoNewWindow -Wait
```

## Distribute your application

Ship the contents of the output folder. No runtime installer is needed. See
[LICENSE.md](../LICENSE.md) for the terms.

## Installers and updates

Applications that update themselves with electron-updater keep working:
the build writes `resources\app-update.yml` beside the executable from
electron-builder's `publish` settings, as electron-builder does, and makes a
Windows installer with the update metadata electron-updater reads
(`latest.yml`, or the channel's file) in `<out>-setup`, or in the folder
`--installer` names. Publish both files where the feed points, and
electron-updater downloads and installs the new version. No `.blockmap` is
written, so each update downloads the whole installer.

Native builds are usually published apart from Electron builds, so give
them their own feed in package.json; it takes the same settings as
electron-builder's `publish`:

```json
"nativeboat": {
  "publish": { "provider": "github", "owner": "me", "repo": "my-app-native" }
}
```

Without it, the native build checks the Electron builds' feed, and the build
warns about it. `"publish": null` turns updates off.

The installer installs for the current user into
`%LOCALAPPDATA%\Programs\<name>`, or into the folder `/D=` names, with Start
menu and desktop shortcuts, an uninstaller and an entry in Installed apps.
It takes `/S` to install silently. An update installs into the folder the
running application is in, after it exits. The installer installs in one
click: the NSIS settings for the license page, the installation folder page
and all-users installs are not used.
