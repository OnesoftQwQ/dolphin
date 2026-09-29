# Dolphin — fork with "folders first within groups"

> This is a personal fork of [KDE/dolphin](https://github.com/KDE/dolphin), based on the
> `release/26.08` branch (Dolphin **26.08.1**). It is **not** affiliated with KDE.
> The upstream README is kept below.

## What this fork changes

Upstream Dolphin has a *Folders First* option and a *Show in Groups* option. When both are
enabled, Dolphin moves **all** folders in front of **all** groups: the list looks like
"all folders grouped by category" followed by "all files grouped by category".

This fork makes *Folders First* apply **inside each group** instead, so every group shows its
own folders first and then its files — the same behavior as Windows Explorer:

```
Upstream (grouped by Modified):        This fork (grouped by Modified):
  ▼ Folders                              ▼ Today
      Documents        Today                 Documents      (folder)
      Downloads        Yesterday             report.pdf     (file)
  ▼ Today                                    photo.png      (file)
      report.pdf                             ...
      photo.png                            ▼ Yesterday
  ▼ Yesterday                                Backup         (folder)
      ...                                    notes.txt      (file)
```

### The setting

A new per-view setting `FoldersFirstWithinGroups` (default **on**) is added. The
*Folders First* menu entry (`View → Sort by → Folders First`) became a submenu with three
mutually exclusive states:

| Menu item | `SortFoldersFirst` | `FoldersFirstWithinGroups` | Result |
|---|---|---|---|
| Do not show folders first | `false` | – | folders and files are mixed |
| Show folders first | `true` | `false` | legacy behavior: folders in front of *all* groups |
| Show folders first within each group **(default)** | `true` | `true` | folders first inside *each* group |

The same choice is available in `View → Adjust View Display Style` (a combo box).

### Implementation

* `src/kitemviews/kfileitemmodel.cpp`
  * `lessThan()` now treats the group as the primary sort criterion and applies
    "folders first" only when both items belong to the same group
    (this is what removes the need to move every folder in front of every group).
  * Added `groupIdentityForItem()` and small helpers (`nameGroupValueForName()`,
    `sizeGroupValueForItem()`, `timeGroupIdentityForDateTime()`) so the comparison uses the
    exact same group boundaries as `nameRoleGroups()`, `sizeRoleGroups()` and
    `timeRoleGroups()`. Size grouping is kept as-is (folders already form their own group).
  * The helpers were extracted from the existing group functions so there is a single source
    of truth.
* `src/settings/dolphin_directoryviewpropertysettings.kcfg` — new boolean
  `FoldersFirstWithinGroups` (default `true`).
* `src/views/viewproperties.{h,cpp}`, `src/views/dolphinview.{h,cpp}`,
  `src/views/dolphinviewactionhandler.{h,cpp}`,
  `src/settings/viewpropertiesdialog.{h,cpp}` — plumbing, the new submenu and the dialog
  combo box.
* `src/tests/kfileitemmodeltest.cpp` — `testFoldersFirstWithinGroups()` verifies the group
  ordering and the fallback to the legacy behavior.
* `po/zh_CN/dolphin.po` — Simplified Chinese translations for the new strings.

### Building

```sh
cmake -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr
cmake --build build -j"$(nproc)"
DESTDIR=/tmp/stage cmake --install build
```

### Fedora Kinoite / rpm-ostree

Prebuilt RPMs (`dolphin` + `dolphin-libs`, built against Fedora 44 / Qt 6.11.2 / KF6 6.30.0)
are attached to the **Releases** of this repository.

```sh
sudo rpm-ostree override replace ./dolphin-*.rpm ./dolphin-libs-*.rpm
systemctl reboot
```

Revert to the official build:

```sh
sudo rpm-ostree override reset dolphin dolphin-libs
systemctl reboot
```

---

*The rest of this file is the upstream Dolphin README.*

# Dolphin

Dolphin is KDE's file manager that lets you navigate and browse the contents of your hard drives, USB sticks, SD cards, and more. Creating, moving, or deleting files and folders is simple and fast. See more information [on Dolphin's homepage](https://apps.kde.org/dolphin/).

![Screenshot](https://cdn.kde.org/screenshots/dolphin/dolphin.png)

## User Documentation

See https://userbase.kde.org/Special:myLanguage/Dolphin.

## Contributing

Like other projects in the KDE ecosystem, contributions are welcome from all. This repository is managed in [KDE Invent](https://invent.kde.org/system/dolphin), our GitLab instance.

* Want to contribute code? See the [GitLab wiki page](https://community.kde.org/Infrastructure/GitLab) for a tutorial on how to send a merge request.
* Reporting a bug? Please submit it on the [KDE Bugtracking System](https://bugs.kde.org/enter_bug.cgi?format=guided&product=dolphin). Please do not use the Issues
tab to report bugs.
* Is there a part of Dolphin that's not translated? See the [Getting Involved in Translation wiki page](https://community.kde.org/Get_Involved/translation) to see how
you can help translate!

If you get stuck or need help with anything at all, head over to the [KDE New Contributors room](https://go.kde.org/matrix/#/#kde-welcome:kde.org) on Matrix. For questions about Dolphin, please ask in the [KDE File Management room](https://go.kde.org/matrix/#/#kde-fm:kde.org). See [Matrix](https://community.kde.org/Matrix) for more details.

## Development Philosophy

Dolphin is a file manager focusing on usability. When reading the term Usability people often assume that the focus is on newbies and only basic features are offered. This is not the case; Dolphin is quite full-featured, but the features are carefully chosen so as to not impede any of the users in the target user groups.

### Target User Groups

Focusing on usability means that features are discoverable and efficient to use. The feature set is defined indirectly by the target user group of Dolphin:

- **Lisa**: Lisa has been familiar with computers for 10 years. From her job, she has experience with Word, Excel and Outlook. At home she mainly uses the computer for browsing the web and writing e-mails. She requires a file manager for managing photos from the camera, documents she gets via e-mail, or PDFs she downloads with a browser. Lisa knows concepts like folders and a file hierarchy, but she is not familiar with the file hierarchy of Linux.

- **Simon**: Simon has been a developer at a software company for 8 years. At home he uses a file manager to maintain his large collection of photos and music. Additionally he owns a small homepage and needs to transfer updated files on the FTP server. Moving and copying files are regular tasks in Simon's workflow.

Not part of the target user group of Dolphin are Fred and Jeff:

- **Fred**: Fred is 75 years old and is able to write e-mails and browsing the web. He is not familiar with file hierarchies and stores all his documents on the desktop.

- **Jeff**: Jeff is Linux-freak since the age of 16 a few years ago. He is a developer and in his spare time he acts as administrator for a small company. Jeff has two monitors to keep the overview about his huge number of opened applications.

This does not mean that Fred or Jeff cannot work with Dolphin. But there might be features and concepts of Dolphin that overburden Fred. Also Jeff might miss some features which are a must-have for his daily work. This is acceptable; there are other tools that cater specifically to their needs.

### Non-Intrusive Features

Before a feature is added in Dolphin, check whether the feature is mandatory for the target user group. If this is not the case, then this does not mean that the feature cannot be added; first it must be clarified whether the feature might be non-intrusive, so that it adds value for users outside the primary target user group of Dolphin. The term "non-intrusive" is mainly related to the user interface. A feature that adds a lot of clutter to the main menu, context menus or toolbar might harm the target user group. In this case the feature should not be added.

A good example of a feature that is non-intrusive is the embedded terminal in Dolphin. It only requires one entry inside a sub-menu, but adds great value for Jeff, who is not part of the target user group.

### Options

Options are mandatory as the "average Joe" user does not exist. Still it is not the goal of Dolphin to offer options for all kind of things. Again the focus is on the possible needs of the target user group. Each additional option makes it harder finding other options, so the same rules for features are applied to options too.
