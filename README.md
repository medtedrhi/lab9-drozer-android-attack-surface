# Android Drozer Attack Surface Lab

## 1. Context

This lab was performed in a defensive and authorized Android security audit context.

The objective is to analyze the attack surface of an intentionally vulnerable Android application using Drozer.  
The analysis focuses on exported Android components, permissions, providers, services, receivers, activities, and manifest configuration.

No real user data was used or extracted.

---

## 2. Lab Environment

- Host system: Windows
- Android environment: Android Studio Emulator
- Drozer Agent: running inside the emulator
- Drozer Console: running from Docker
- Target application: AndroGoat
- Target package: `owasp.sat.agoat`

---

## 3. Drozer Agent Started on Emulator

The Drozer Agent application was opened inside the Android emulator.  
The Embedded Server was enabled to allow the Drozer console to communicate with the emulator.

<img width="397" height="852" alt="image" src="https://github.com/user-attachments/assets/7bdeaf35-e1ab-45d7-ab01-45446f4f5502" />


---

## 4. ADB Port Forwarding

ADB forwarding was configured to connect the Drozer console to the Drozer Agent running inside the emulator.

Command used:


adb forward tcp:31416 tcp:31415
adb forward --list

The forwarding maps the host port 31416 to the emulator Drozer Agent port 31415.

<img width="1424" height="578" alt="image" src="https://github.com/user-attachments/assets/3c0c39e5-9b34-40ab-abad-47f7f23a0403" />


5. Drozer Console Connection

The Drozer console was started and connected to the Drozer Agent.

Command used:

drozer console --server host.docker.internal:31417 connect

After the connection, the Drozer prompt appeared:

dz>

This confirms that the Drozer console can communicate with the Drozer Agent.

<img width="1032" height="603" alt="image" src="https://github.com/user-attachments/assets/89c1fdb4-ce2e-4cfa-a094-5a2cad7907db" />


6. Device Validation

The following Drozer module was executed to collect device information:

run information.deviceinfo

Result:

Exception occured: /proc/version: open failed: EACCES (Permission denied)

This result shows that Drozer is connected, but the Android emulator restricts access to /proc/version.
This restriction does not prevent the analysis of installed packages or Android components.

<img width="954" height="147" alt="image" src="https://github.com/user-attachments/assets/fcc145e8-e06e-4592-bf1f-19e0d5f86dd0" />


7. Target Package Identification

The target application package was identified using Drozer.

Command used:

run app.package.list -f goat

Target package identified:

owasp.sat.agoat

<img width="748" height="79" alt="image" src="https://github.com/user-attachments/assets/68e30bf3-994d-4686-8a6b-aae9a73e5c12" />


8. Package Information

General information about the target package was collected.

Command used:

run app.package.info -a owasp.sat.agoat

This command provides package metadata and helps confirm that the correct application is being analyzed.

<img width="1043" height="543" alt="image" src="https://github.com/user-attachments/assets/41da41b7-4fc2-4fc9-9782-c6f2ef759a92" />


9. Attack Surface Summary

Drozer was used to summarize the attack surface of the target application.

Command used:

run app.package.attacksurface owasp.sat.agoat

This command shows the number of exposed components such as:

Activities
Services
Broadcast Receivers
Content Providers
Debuggable status

<img width="727" height="189" alt="image" src="https://github.com/user-attachments/assets/1ebf5d75-cc6d-4e23-8c40-d64e4440075a" />


10. Exported Activities Analysis

Activities were analyzed to identify components that may be accessible from outside the application.

Command used:

run app.activity.info -a owasp.sat.agoat

Security risk:

Exported activities can be launched by other applications.
If an exported activity displays sensitive screens or does not verify the user state, it may allow unauthorized access or bypass the normal application flow.

Evidence file:

preuves/activities/exported_activities.txt

<img width="816" height="174" alt="image" src="https://github.com/user-attachments/assets/ad681b29-c3ac-490c-b069-8ffe78ed7cfa" />


11. Exported Services Analysis

Services were analyzed to identify exported background components.

Command used:

run app.service.info -a owasp.sat.agoat

Security risk:

Exported services may allow another application to trigger internal background functionality.
If no permission or input validation is used, this may lead to unauthorized execution of internal operations.

Evidence file:

preuves/services/exported_services.txt

<img width="744" height="128" alt="image" src="https://github.com/user-attachments/assets/a2bc4025-f81f-4a48-9ee1-9f47ec961151" />


12. Broadcast Receivers Analysis

Broadcast receivers were analyzed to identify receivers exposed to external intents.

Command used:

run app.broadcast.info -a owasp.sat.agoat

Security risk:

Exported broadcast receivers may receive intents from other applications.
If the receiver does not validate the received action or sender, it may trigger unintended application behavior.

Evidence file:

preuves/receivers/exported_receivers.txt

<img width="684" height="175" alt="image" src="https://github.com/user-attachments/assets/f4bb19ea-64a1-4437-ab6b-58dff7a736d7" />


13. Content Providers Analysis

Content providers were analyzed to identify exposed data interfaces.

Command used:

run app.provider.info -a owasp.sat.agoat

Security risk:

Content providers can expose application data through content URIs.
If a provider is exported without read or write permissions, other applications may access or modify application data.

Evidence file:

preuves/providers/exported_providers.txt

<img width="867" height="278" alt="image" src="https://github.com/user-attachments/assets/17fbb54e-6754-40f9-9a7a-23ffab6f1bda" />


14. Android Manifest Analysis

The Android manifest was reviewed using Drozer.

Command used:

run app.package.manifest owasp.sat.agoat

The manifest analysis focused on:

Exported components
Declared permissions
Intent filters
Activities
Services
Broadcast receivers
Content providers
Read and write permissions

Evidence file:

preuves/manifest/manifest_analysis.md

<img width="1527" height="984" alt="image" src="https://github.com/user-attachments/assets/9c54bf3d-f8d5-434c-b87b-dd229e4e11eb" />
<img width="1599" height="1001" alt="image" src="https://github.com/user-attachments/assets/50d946c7-c0c3-4730-aa9f-94a03f592737" />
<img width="1469" height="1035" alt="image" src="https://github.com/user-attachments/assets/d083de14-59c9-4912-8ed0-72e5b68ed60e" />
<img width="1917" height="995" alt="image" src="https://github.com/user-attachments/assets/73e78c8d-3276-4262-8932-2bdf536ae9ca" />


15. Provider URI Discovery

Drozer was used to search for referenced content provider URIs.

Command used:

run scanner.provider.finduris -a owasp.sat.agoat

No sensitive data was extracted.
The objective was only to identify potential provider URIs and evaluate whether they are exposed.

<img width="1256" height="370" alt="image" src="https://github.com/user-attachments/assets/20f6b758-f52f-4e27-9772-f284c6720c4a" />


16. Evidence Folder

The collected evidence was organized in the following folder structure:

preuves/
├── activities/
│   └── exported_activities.txt
├── services/
│   └── exported_services.txt
├── receivers/
│   └── exported_receivers.txt
├── providers/
│   └── exported_providers.txt
└── manifest/
    └── manifest_analysis.md

Screenshot:
