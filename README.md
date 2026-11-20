CALLRECORDER FP5 — INSTALLATION ON A CLEAN UBUNTU TOUCH 24.04 SYSTEM
====================================================================
Variant: /ril_0, 16 kHz mono, voicecall-record profile
Document prepared: 16 November 2026
Status: experimental version, confirmed in a call test on one Fairphone 5.

1. WHAT IS IT?
--------------
CallRecorder FP5 is a simple Bash script that automatically detects answered
cellular calls through oFono, switches the PulseAudio profile to
voicecall-record, records audio using parecord, and saves the result as a WAV
file. When the call ends, it stops recording and restores the audio profile.
It runs as a per-user systemd service (no sudo required).

This package is NOT the original callrecorder-ota.2. It is an independent,
experimental adaptation based on testing with a Fairphone 5 running Ubuntu
Touch 24.04. The original archive is not needed.

IMPORTANT: This is the version that successfully recorded calls after the
modem was set to /ril_0. It does NOT yet add phone numbers to filenames: an
earlier attempt at that modification interfered with automatic recording.

2. PACKAGE CONTENTS
-------------------
recorder.sh             — main script for detecting and recording calls.
callrec-fp5.service     — per-user systemd service definition.
install.sh              — installer (no sudo; backs up an existing version).
uninstall.sh            — uninstaller (does not delete recordings).
README-PL.txt           — full Polish description and instructions.
INSTRUKCJA-SZYBKA.txt   — short Polish Terminal instructions.

This separate English document is provided as an additional guide. If using
the bilingual package, it is also included as README-EN.txt.

3. REQUIREMENTS AND COMPATIBILITY
----------------------------------
- Fairphone 5 with Ubuntu Touch 24.04 (tested on one user's device).
- Access to oFono on the system D-Bus: org.ofono.
- Commands: bash, dbus-send, pactl, parecord, systemctl, and basic utilities.
- PulseAudio profile voicecall during a call and voicecall-record for recording.
- Audio source: source.primary_input; audio card: droid_card.primary.
- Active modem: /ril_0. This matters: on the tested phone, automatic selection
  of the first listed modem returned /ril_1 and recordings were not started.

On another Fairphone 5, with a different SIM setup or port version, device
names and the ability to capture calls may differ. Successful recording on
every device cannot be guaranteed.

4. CLEAN INSTALLATION — STEP BY STEP
-------------------------------------
A. Install the Terminal app from OpenStore if it is not already installed.
B. Download callrec-fp5-ut24-sprawdzony-ril0.tar.gz to the phone.
C. Place it in Downloads. In Terminal, run:

   cd ~/Downloads
   tar -xzf callrec-fp5-ut24-sprawdzony-ril0.tar.gz
   cd callrec-fp5-ut24-sprawdzony-ril0

D. Check the required commands and oFono access:

   bash ./recorder.sh --check

   You should see a message containing:
   "OK: oFono odpowiada; modem=/ril_0" (Polish: oFono responds).
   Outside a call, the audio profile is normally default.
   If the check fails, DO NOT run the installer.

E. Install and enable automatic startup:

   bash ./install.sh

   Do not use sudo. The installer writes only to user directories.
   If an earlier version exists, a backup is stored in Downloads.

F. Check the service:

   systemctl --user is-active callrec-fp5.service
   systemctl --user is-enabled callrec-fp5.service

   Expected outputs are usually: active and enabled.
   NOTE: active only means that the service is running; it does NOT prove
   that calls are being recorded.

G. With the other person's consent, make a short test call (15–30 seconds).
   After hanging up, run:

   ls -lt ~/MyRec | head

   Open the new WAV file and CHECK that both participants can be heard.
   Then reboot the phone and repeat the test.

5. HOW RECORDING WORKS
----------------------
- The script polls oFono every few seconds for call state on /ril_0.
- It starts recording answered calls in the active or held state when the
  active audio profile is voicecall.
- It switches to voicecall-record and starts parecord.
- parecord settings: 16,000 Hz, one channel (mono), WAV container.
- It first creates a temporary .wav.partial file and, after recording ends,
  renames it to .wav.
- Unanswered calls are not recorded.
- The first seconds of a call may be missed while the script detects its state.
- Recordings are stored locally; the program does not upload or automatically
  delete them.

6. RECORDING LOCATION AND FILENAMES
------------------------------------
Recording folder: /home/phablet/MyRec (that is, ~/MyRec).
Example filename: 2026-11-16_20-45-30_12345.wav
Meaning: date_time_random-number.wav

NOTE: The currently tested version does NOT put the phone number in the name.
That feature would require a separate, properly tested update.

The MyRec directory is assigned permissions 700, and the script uses
umask 077, so new recordings are private to the current user by default.
WAV files can consume considerable storage; monitor your available space.

7. MANAGING THE SERVICE
-----------------------
Show status:
   systemctl --user status callrec-fp5.service

Stop the service until it is started again:
   systemctl --user stop callrec-fp5.service

Start the service:
   systemctl --user start callrec-fp5.service

Disable automatic startup and stop it:
   systemctl --user disable --now callrec-fp5.service

Re-enable automatic startup and start it:
   systemctl --user enable --now callrec-fp5.service

View recent logs:
   journalctl --user -u callrec-fp5.service -n 50 --no-pager

Check oFono and dependencies without recording:
   bash ~/.local/share/callrec-fp5/recorder.sh --check

Do not run two instances of recorder.sh at the same time.

8. UNINSTALLATION
-----------------
Do not uninstall while a call is in progress. From the extracted package
folder, run:

   bash ./uninstall.sh

This removes the service and script but PRESERVES ~/MyRec and its recordings.

9. TROUBLESHOOTING
------------------
A. Service is active, but no WAV file appears:
   - Confirm that the phone uses modem /ril_0.
   - Check oFono: bash ~/.local/share/callrec-fp5/recorder.sh --check
   - View recent log entries:
     journalctl --user -u callrec-fp5.service -n 80 --no-pager
   - Check whether a .wav.partial file remains in ~/MyRec.
   - Do not immediately change audio settings: on the tested Fairphone 5,
     the issue was selecting /ril_1 instead of /ril_0.

B. Call audio does not work properly after a failure:
   - End the call and check the current profile:
     LC_ALL=C pactl list cards | grep 'Active Profile:'
   - Normally it should be default once the call has ended.
   - If audio issues persist, stop the service and reboot the phone.

C. pactl, parecord, or dbus-send is missing:
   - Installation will be aborted.
   - Do not remount the system partition as writable or install packages
     using apt without first evaluating Ubuntu Touch compatibility.

D. Calls made with a second SIM (/ril_1):
   - This package intentionally targets /ril_0 because that is what worked
     on the tested Fairphone 5. Calls handled by a different modem may be
     missed.
   - Supporting another modem requires a deliberate change to call detection
     and a separate test.

E. "is-active=active" is not proof of successful recording. Listen to a new
   WAV file from an actual phone call to verify recording works.

10. SECURITY AND PRIVACY
------------------------
- Call recording may be subject to laws and notification requirements.
  Obtain consent and follow the rules that apply to you.
- The script can access information about active calls and the microphone.
- Store the files securely, protect the phone, and make backups as needed.
- Switching to voicecall-record may interfere with call audio on other
  software versions. Test the setup before relying on a recording.
- This is an unofficial, experimental solution with no guarantee on other
  Ubuntu Touch versions and no official UBports support.

11. FILES INSTALLED IN THE USER ACCOUNT
---------------------------------------
~/MyRec/
~/.local/share/callrec-fp5/recorder.sh
~/.config/systemd/user/callrec-fp5.service

It does NOT modify /usr, /etc, or the system partition and does NOT require
sudo.

12. TECHNICAL NOTES
-------------------
This variant preserves the code of the earlier recorder and replaces one
modem-selection line with `modem=/ril_0`. The user confirmed that this change
restored automatic recording of both call participants and that calls were
saved as WAV files. It has NOT been tested on a fresh Ubuntu Touch 24.04
installation on another phone: verify the result as described in section 4G.

Further references:
Audio profile implementation:
https://github.com/FuriLabs/pulseaudio-modules-droid-modern
Ubuntu Touch audio documentation:
https://docs.ubports.com/gl/latest/porting/configure_test_fix/Sound.html
Per-user systemd service documentation:
https://www.freedesktop.org/software/systemd/man/latest/systemd.special.html
Polish version: README-PL.txt; additional documentation inside folders.
