GTK Icon Guardian

Automatically restore the missing image-missing.png icon in GTK themes to prevent application crashes.

------------------------------------------------------------------------------
The Problem
------------------------------------------------------------------------------

- GTK applications use image-missing.png as a fallback for missing icons.
- If this file is missing or replaced by a 1x1 transparent PNG, GTK can crash,
  causing login loops or application failures.
- This is a known issue with some icon themes (e.g., Adwaita in Fedora).

------------------------------------------------------------------------------
The Solution
------------------------------------------------------------------------------

This tool permanently protects your system by:

- Detecting the real icon (searching all themes like AdwaitaLegacy, gnome, Mint-Y, etc.)
- Creating a backup of the real icon.
- Installing a boot-time restore service (before the display manager starts).
- Running a watchdog monitor (using inotify) that restores the icon instantly if deleted.
- Setting up auditd forensic logging (optional) to track who deleted the file.
- Adding a daily health-check timer as a safety net.
- Adding a periodic check timer (every 5 minutes) that sends alerts if restoration fails.

------------------------------------------------------------------------------
Requirements
------------------------------------------------------------------------------

- Fedora/RHEL (or any distribution with dnf – adapt for apt if needed)
- Python 3.6+
- sudo privileges

The script will automatically install:
- audit, audit-libs (for logging)
- inotify-tools (for the bash monitor)
- python3-watchdog (fallback, but the bash monitor is used)

------------------------------------------------------------------------------
Installation
------------------------------------------------------------------------------

Clone the repo and run the installer:

    git clone https://github.com/swipswaps/gtk-icon-guardian.git
    cd gtk-icon-guardian
    sudo python3 gtk_guardian.py

That's it. The entire system is self-healing from now on.

------------------------------------------------------------------------------
What Happens
------------------------------------------------------------------------------

- The script searches all icon themes for a real image-missing.png (not 1x1).
- Copies it to all Adwaita and hicolor status directories.
- Saves a master backup to /usr/local/lib/gtk-icon-backup/.
- Sets up auditd (if possible) to log deletions.
- Installs a bash-based watchdog monitor using inotifywait.
- Installs a systemd service for the monitor (auto-restarts on failure).
- Installs a systemd service for boot-time restoration (runs before login).
- Installs a daily systemd timer for extra safety.
- Installs a periodic check timer (every 5 minutes) that tries to restore the icon
  and sends an alert if it fails.

------------------------------------------------------------------------------
Verify It Works
------------------------------------------------------------------------------

Delete the icon and watch it come back:

    sudo rm /usr/share/icons/Adwaita/16x16/status/image-missing.png
    sleep 2
    ls -la /usr/share/icons/Adwaita/16x16/status/image-missing.png

Check logs:

    sudo tail -f /var/log/gtk-icon-restore.log

To see forensic logs (who deleted it):

    sudo ausearch -k gtk-icon-watch -ts today | grep -A10 image-missing.png

------------------------------------------------------------------------------
Uninstall
------------------------------------------------------------------------------

To remove all components:

    sudo systemctl stop gtk-icon-monitor.service gtk-icon-boot-restore.service gtk-icon-check.timer
    sudo systemctl disable gtk-icon-monitor.service gtk-icon-boot-restore.service gtk-icon-healthcheck.timer gtk-icon-check.timer
    sudo rm -f /usr/local/bin/gtk-icon-* /usr/local/bin/gtk_guardian.py
    sudo rm -rf /usr/local/lib/gtk-icon-backup

------------------------------------------------------------------------------
License
------------------------------------------------------------------------------

MIT
