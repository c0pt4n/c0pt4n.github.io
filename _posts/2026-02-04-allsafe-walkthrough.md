---
title: Allsafe Android Walkthrough
date: 2026-02-04 23:08:20 +0200
categories: [apk]
tags: [mobile-pentest, android]
image:
  path: /assets/img/allsafe/allsafe-android-walkthrough.png
---

Hello guys, today we are going to solve `Allsafe` application, let's start by looking at the first challenge.

## 01 - Insecure Logging

![](/assets/img/allsafe/allsafe-android-walkthrough-1.png)

As the name of the challenge reveals, it says there is `insecure logging` so let's check the logs of the application and try entering secrets...

```bash
adb shell pgrep allsafe
```

![](/assets/img/allsafe/allsafe-android-walkthrough-2.png)

![](/assets/img/allsafe/allsafe-android-walkthrough-3.png)

> As shown here in the log, the credentials are shown in the logs!

---

## 02 - Hardcoded Credentials

![](/assets/img/allsafe/allsafe-android-walkthrough-4.png)

To get the ActivityName, we use the following command:

```bash
adb shell dumpsys | grep -i "currentfocus"
```

![](/assets/img/allsafe/allsafe-android-walkthrough-5.png)

Let's fire up **Jadx** and check what can we reach

By inspecting the activity of hardcoded credentials, we find out that there is a hardcoded credential in it

![](/assets/img/allsafe/allsafe-android-walkthrough-8.png)

Also there is a point to a variable named `dev_env` in `strings.xml` so let's check what can we reach.

![](/assets/img/allsafe/allsafe-android-walkthrough-7.png)

![](/assets/img/allsafe/allsafe-android-walkthrough-6.png)

The first dev_env is found, now we found the two credentials.

---

## 03 - Firebase Database

![](/assets/img/allsafe/allsafe-android-walkthrough-9.png)

By statically analyzing the code, we find out that there is a hardcoded firebase URL.

![](/assets/img/allsafe/allsafe-android-walkthrough-10.png)

Let's try checking if there is a misconfiguration there..

![](/assets/img/allsafe/allsafe-android-walkthrough-11.png)

And we found the misconfiguration!

---

## 04 - Insecure Shared Preferences

![](/assets/img/allsafe/allsafe-android-walkthrough-12.png)

Looking here, we find it stores the data in SharedPreferences

![](/assets/img/allsafe/allsafe-android-walkthrough-13.png)

Let's fire up MtManager and check what we reach.

![](/assets/img/allsafe/allsafe-android-walkthrough-14.png)

![](/assets/img/allsafe/allsafe-android-walkthrough-15.png)

And it is done!

---

## 05 - SQL Injection

![](/assets/img/allsafe/allsafe-android-walkthrough-16.png)

As written we can do it without reverse engineering, so we can try to inject.

Let's try a simple payload and check.

```SQL
test' or '1'='1'--
```

![](/assets/img/allsafe/allsafe-android-walkthrough-17.png)

and it returned all data!

---

## 06 - BIN Bypass

![](/assets/img/allsafe/allsafe-android-walkthrough-18.png)

As shown in the hint, we will use Frida.

using Gemini, I wrote a code to bypass PIN

```javascript
// Allsafe PIN Validation Bypass
Java.perform(function () {
  const PinBypassFragment = Java.use('infosecadventures.allsafe.challenges.PinBypass');
  PinBypassFragment.checkPin.implementation = function (pin) {
    return true; // Bypass the check by always returning true
  };
});
```

and now run frida

```bash
frida -U -f infosecadventures.allsafe -l ./frida-pin-bypass.js
```

![](/assets/img/allsafe/allsafe-android-walkthrough-19.png)

It hooked successfully! now let's try any pin as 1111

![](/assets/img/allsafe/allsafe-android-walkthrough-20.png)

and BOOM! Access is granted successfully.

---

## 07 - Root Detection

![](/assets/img/allsafe/allsafe-android-walkthrough-21.png)

In this challenge we should bypass the root detection, but firstly let's inspect the static code.

![](/assets/img/allsafe/allsafe-android-walkthrough-22.png)

