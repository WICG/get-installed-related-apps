# Get Installed Related Apps API

# Abstract
As the capabilities of the web grow, the functionality of web apps begins to
match that of corresponding native apps. The situation of users having a web
app and the corresponding native app both installed on the same device will
become more common, and the feature sets of these apps will converge.

It is important to allow websites to detect if an app is installed, whether a
native app or a web app, to allow them to disable functionality that should be
provided by the other app.

The GetInstalledRelatedApps API allows web apps to detect if related apps are
installed on the current device.

# Querying the installed local apps that specify the website.

From an `async` function:

```js
const listOfInstalledApps = await navigator.getInstalledRelatedApps();
for (const app of listOfInstalledApps) {
  // These fields are specified by the Web App Manifest spec.
  console.log('platform:', app.platform);
  console.log('url:', app.url);
  console.log('id:', app.id);

  // This field is provided by the UA.
  console.log('version:', app.version);
}
```

# Describing a relationship from an application to website (and vice versa)
This API is being developed with the assumption that a system exists to create
associations from installed applications to websites.

We can define relationships between a website and other applications by
using the "related_applications" member of the web application manifest.

Example:
```json
{
  "related_applications": [
    {
      "platform": "play",
      "url": "https://play.google.com/store/apps/details?id=com.example.app1",
      "id": "com.example.app1",
      "min_version": "2",
      "fingerprints": [
        {
          "type": "sha256_cert",
          "value": "92:5A:39:05:C5:B9:EA:BC:71:48:5F:F2"
        }
      ]
    },
    {
      "platform": "itunes",
      "url": "https://itunes.apple.com/app/example-app1/id123456789"
    },
    {
      "platform": "webapp",
      "url": "https://example.com/manifest.json"
    }
  ]
}
```

Each platform has its own method of verifying a relationship. In Android, the
[Digital Asset Links](https://developers.google.com/digital-asset-links/v1/create-statement)
system can be used to define an association between a website and an application.
If the application is installed locally and defines an association with the
requesting web application, we return the app as defined in the
"related_applications" member.

# Privacy Considerations
This feature only works with sites using HTTPS. This ensures that the website
cannot be spoofed, and that the association between the site and application is
valid.

The association between the web app and its counterpart is bidirectional,
meaning that the web app has to declare its association with the related app,
and the related app has to declare its association with the web app. This
prevents malicious websites from fingerprinting users and getting a list of
their installed applications.

User Agents should also take timing side channel attacks into account in their
implementation, to avoid potential leaking of information about installed
applications on a user's device. To remove the timing side channel,
implementations could pursue several options. One is to delay resolution of the
API call by a random, *but fixed per app*, delay. As long as the caller cannot
predict, guess, or otherwise determine what the delay was, nor re-query many
times to filter out the delay 'noise', the caller gains no timing side-channel
information about the installation status of the app.

User Agents may also limit the number of related apps checked to limit the
amount of fingerprinting information exposed to websites.

The User Agent must not return installed applications when running in a privacy
preserving mode, for example Incognito in Chrome or Private Browsing in Firefox.

# Abuse Considerations
For websites that own native applications that generally come pre-installed on
a platform, this API can be used as a privacy mode detector if the native app
is found to be missing (albeit with a non-negligible false positive rate).
User Agents are encouraged to keep track of how this API is being used,
specifically for the few origins that own the native apps that are
pre-installed on a specific platform.

Another abuse concern is that web applications might use this API to drive
users away from a web environment, to the native counterpart. Although this
practice already exists in the wild, User Agent should be wary of this
potential abuse vector. Note that this is why no installed applications must be
returned when running in in a privacy preserving mode.
