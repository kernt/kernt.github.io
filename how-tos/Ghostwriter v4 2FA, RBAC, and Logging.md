---
tags:
  - 2fa
  - rbac
  - logging
  - ghostwriter
  - ipam
  - ISO27001
---
**Ghostwriter v4 is officially here!** Technically, it’s been available as a release candidate for a while, but we have arrived at its final release. This major release focuses on something important to Ghostwriter users: identity and access management (IAM)!
# Role-Based Access Controls

Ghostwriter 3, released in June 2023, introduced the GraphQL API and the first iteration of role-based access controls (RBAC). Ghostwriter 3 applied RBAC only to the GraphQL API, as we didn’t want to hold back the GraphQL API while converting the web interface to support RBAC.

With Ghostwriter 4, the access controls apply to every webpage and view. Ghostwriter uses three roles to determine what you can see and access: _admin_, _manager_, and _user_. The _manager_ role grants the same level of access that every user has in Ghostwriter version <=v3.2. Accounts with this role can do anything except access the admin console. The _user_ role considerably narrows the scope of what is available to a user with this role.

Accounts with the _user_ role can only access client and project data if a _manager_ has assigned that person to the project. Otherwise, this role has the standard permissions you might expect. They can edit or delete their comments, update their profiles, and view the shared information in the various libraries (e.g., findings, domains, etc.).

These limitations mean an account can’t immediately view every client or project tracked in Ghostwriter. Not only does this help you manage access to client and project data, but certifications like ISO 27001 and SOC 2 strongly encourage RBAC. If your company has or is seeking these certifications, Ghostwriter v4.0 now complies with these requirements!

You can learn more about RBAC and the authorization model here:
# Two-Factor Authentication

We’re also excited to introduce support for two-factor authentication (2FA)! Previously, you could implement 2FA via a single sign-on (SSO) provider like Azure or Google, but now Ghostwriter 4 supports time-based one-time passwords (TOTP) for its local accounts. 2FA has been on our wishlist for some time and this release felt like the perfect time to roll it out.

To enroll a device, visit your account profile and click the new _Set Up 2FA_ button. Ghostwriter will provide you with a QR code to scan with 1Password, Google Authenticator, or any other TOTP app you prefer. The setup page also provides a secret if you want to configure an app manually.

![](5vLcYqD4Lg_XLUzY.webp)

Administrators can configure accounts to require 2FA before the user can access Ghostwriter. Requiring 2FA is configurable on a per-user basis. With this option enabled, accounts without a valid TOTP device can only log in, log out, change their password, and access the 2FA setup page.

Ghostwriter’s 2FA also supports backup codes. You can generate and retrieve your backup codes from your profile page at any time after configuring 2FA.
# Additional Security Enhancements

We’ve included further enhancements to Ghostwriter 4 to round out the IAM improvements. The previous Ghostwriter releases used Django’s defaults for session management and while these defaults aren’t inherently bad, they permissive for applications storing sensitive information.

The default Django sessions expire after two weeks. To tighten up session management, we’ve added configuration options for three essential values:

- _DJANGO_SESSION_COOKIE_AGE_: Sets the number of seconds a session cookie will last before expiring (default: _32400_ seconds or nine hours)
- _DJANGO_SESSION_SAVE_EVERY_REQUEST_: Sets whether the session cookie will refresh on every request (default: _true_)
- _DJANGO_SESSION_EXPIRE_AT_BROWSER_CLOSE_: Sets whether the session cookie will expire when the browser is closed (default: _true_)

These defaults are a good starting point. Still, you should consider how your team uses Ghostwriter and adjust accordingly. We chose nine hours for the expiration so a login session can last an entire workday — just in case someone is in the middle of something and has to walk away for an extended period. You may want to reduce this value to one or two hours.

With _DJANGO_SESSION_SAVE_EVERY_REQUEST_ set to _true_, the server will update the session with each request. Updates reset the expiration, so a short expiry period won’t log out anyone actively using Ghostwriter but will allow inactive sessions to expire.

If set to _true_, the last option will expire sessions after the browser quits. However, whether the session ends when you close the browser window depends on the browser. Some browsers, like Chrome, will keep sessions active, so you may need to quit or exit the browser to end the session versus just closing the browser window.

You can manage these values via the Ghostwriter command-line interface (CLI) tool.
# REST in Peace

Another significant change in this release is the complete deprecation and removal of the legacy REST API. Ghostwriter v3 kept the REST API to support some utilities that still used it following the introduction of the GraphQL API. One of those utilities was _cobalt_sync_, the tool that syncs Cobalt Strike activity logs to Ghostwriter.

Alongside this release, we also have _cobalt_sync_ v2! Cody Thomas rewrote _cobalt_sync_ from scratch. This new _cobalt_sync_ does away with Aggressor script (the old _oplog.cna_) and parsing Beacon output in favor of easily deployed Dockerized services. These services use the new _cobalt_parser_ utility written in Golang to monitor the beacon.log files on your team server and a lightweight Python web server to send the event information to Ghostwriter.

This change comes with a few benefits. The most significant is the ability to hash Beacon events to prevent logging duplicate events. This hashing and the new Redis service make it simple to resume logging without worrying about missing or duplicating any events. You can even start logging late and _cobalt_sync_ will log everything that has occurred up to that point.

Another benefit is the Beacon log files include events that Aggressor scripts that monitor the console activity will miss. Tasking that happened automatically or programmatically via an Aggressor script will now log to Ghostwriter. You’ll see two events in Ghostwriter for user inputs: the user manually issuing a task and tasks registered with the Beacon. That dual log distinction happens so that _cobalt_sync_ can ensure _everything_ that happens through that Beacon logs to Ghostwriter.
# Other New Features & Enhancements

Ghostwriter v4 is launching with notable quality-of-life improvements from community requests. You can now create project-specific points of contact. Previously, you could only set points of contact for a client, and all contacts were automatically part of any projects attached to that client. Now, you can assign client points of contact to your projects or create new points of contact that are only part of that individual project.

You can also mark a contact as the “primary” contact. Ghostwriter adds the primary contact to a special _recipient_ key in the report data. If your reports put contact information on the cover page, you can now use _recipient_ to add those details wherever needed.

The last addition is a new option to copy an activity log entry to your clipboard as JSON. Once clicked, Ghostwriter serializes the log entry into a JSON object and copies it to your clipboard for easy sharing.

```json
[
  {
    "Start Date":"2023–09–13 17:59:21",
    "End Date":"2023–09–13 17:59:21",
    "Source":"getghostwriter.io",
    "Destination":"ghostwriter.wiki",
    "Tool Name":"beacon",
    "User Context":"GW\\Benny",
    "Command":"./autopwn pew pew -i",
    "Description":"",
    "Output":"",
    "Comments":"",
    "Operator":"cmaddalena",
    "Tags":"att&ck:T1059"
  }
]
```

This is helpful when you want to share individual log entries with a client or another team while discussing an event.
# Wrap Up

You can get Ghostwriter v4.0.0 right now! Go here to view the release:

You can upgrade an existing server with by running `git pull` to fetch the latest code. Then bring down any running containers and build v4. As always, be responsible and take a snapshot of your server before upgrading in production