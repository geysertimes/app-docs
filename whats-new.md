# GeyserTimes for Android: what's new in 5.2.0

2026-11-02, updated 2026-11-17 · J. Alan Glennon, PhD · v5.2.0 Lead · GeyserTimes: @glennon  
GeyserTimes for Android Team: Altstidl, Boekel, Glennon, and Young

## Summary

Version 5.2.0 updates the app with modern Android tooling and best practices while retaining the current Android app's existing look and feel. The changes are most significant to internal architecture: the code was converted from Java to Kotlin, the screen design from XML layouts to Jetpack Compose, and the on-device database from Realm to Room. The update was designed to minimize user experience disruption, and if things work as hoped, the process should be minimally painful. Changes to UI/UX are small, with the exception of a change to the flagging system and a redesigned Webcam screen. The app allows users to flag an eruption or note once, and double flagging is largely disallowed. The app possesses an off-line first philosophy, in consideration of Yellowstone's spotty and unreliable Internet connectivity.

## Development note about the use of artificial intelligence

The update from v.4.1.3 to v5.2.0 originally was undertaken manually by me (glennon) starting in late August 2026--using Gemini in Android Studio to assist in explaining code elements. A large amount of the Java to Kotlin conversion was conducted in this manner as well as most of the conversion from XML Layouts to Jetpack Compose. This took about two weeks. When I began working on the Realm to Room conversion, the effort became quite difficult for me to do, so I began using Gemini more liberally. Unfortunately, both the most-recent Gemini Flash and Pro versions introduced an endless number of bugs, and there were many false starts. These failures spanned about a week in themselves. I then abandoned Google Gemini Pro, and began using Anthropic Claude Opus 5.5 with Extra High thinking. It was well suited for the task, and over a day, it designed a Realm to Room conversion plan and eventually a more comprehensive app update plan. From there, I "supervised" the  plan's implementation by examining its code and testing it both automatically and manually after each completed task. Claude was soon performing apparently excellent work on each task, and my supervision lessened. As such, the updated app has largely been created by Claude Opus 5.5. To reinsert a human in the loop, a full-app audit and testing plan was devised for the near-completed app. Manual testing of 108/108 workflows passed, including the upgrade from v4.1.3 to v5.2.0 (rehearsed on the Moto G against the test server, and made through Google Play on the Pixel 10a) and a side-by-side comparison of v5.2.0 on the Moto G and the Pixel 10a. Two manual tests on the original checklist of 111 were replaced by automated tests because of their complexity: a database clean-up started during a sync, and times entered on the days the clocks change for daylight saving time. One manual test could not easily be reproduced by hand, a notification prompt that v4.1.3 leaves for someone who never answered it.

## At a glance

| | v4.1.3 (Google Play, May 2023) | 5.2.0 (in internal testing on Google Play since 2026-11-12; called 5.1.0 during its development) |
|---|---|---|
| Programming language | Java (154 files) | Kotlin (188 files), Google's preferred language for Android |
| Screens | XML layouts (56) in 17 activities, the full-screen windows Android manages | Jetpack Compose, which builds screens in Kotlin code. One activity, MainActivity, holds all the main screens, Submit included, and switches between them much as fragments did. Startup and setup, Log in and Account keep their own activities: 4 in all |
| On-phone database | Realm 10.15 | Room, Android's own database library. Realm is kept only to copy v4.1.3's data once, on the first launch after the update |
| Android versions | 5.0 and newer, built for Android 13 | 7.0 and newer, built for Android 17 |
| Look | Original colors | Same layout and data display, with modest use of Compose blue (#00639B) to tell the new version apart. Geyser field colors were changed to maximize color separation of non-Yellowstone fields (e.g., Chile, Iceland, and New Zealand) |
| GeyserTimes server | API v5 | API v5, unchanged |
| Automated tests | None | 950 unit tests, run before every commit and by GitHub on every push, and 3 on-device tests |
| Package size, all phone types | 19.8 MB | 40.6 MB |
| Download from Google Play, new install | About 9.5 MB (estimate) | 8.24 MB (Google Play's figure); about 5.6 MB once Realm is removed (estimate) |
| Version code | 102 | 303 |

Package sizes are measured from release builds of each version. The 5.2.0 package is bigger because it stores Realm's native library (6 to 10 MB for each of four processor types) uncompressed, as current Android tools do, so the phone can use it without unpacking it. Google Play compresses the download and sends each phone only its own processor's copy, so downloads stay about the same. The estimates are for a typical phone (64-bit ARM); 5.2.0's own figure is the one Google Play gave for its first internal testing release.

The version code is the number Google Play compares to decide which build is newer. A v4 release can take any number from 103 to 300 and still update to 5.2.0.

## Improvements

**Spotty signal in the park.** The app is used mostly where there may be no signal for hours, so offline is treated as the normal case. Efforts were made to bolster existing function on the following:

- Entries made offline are never lost or hidden. They show in teal until uploaded, and moving the first date later never deletes them (Clean database does, after a warning).
- A download cut off by weak signal resumes where it stopped.
- Network requests now have time limits suited to weak signal (30 seconds to connect, 60 to read; v4.1.3 set none), and retries wait 30 seconds, then twice as long each time, instead of every 10, 20, 30 seconds and so on.
- Coming back to the app fetches new reports, unless it synced in the last two minutes. Pushes from the server currently don't arrive (see the last section), so without this the app only updated on a pull to refresh.
- A change the server refuses stays on the phone instead of going up again at every sync. The Timeline lists it once, with Discard and Keep, and the entry's page offers "Discard my change". The server will soon refuse uploads without a login (geysertimes-dot-org issue #1408), and asks apps to work this way.

**Sync less likely to get stuck.** In v4.1.3, one downloaded record the app couldn't read sometimes made the whole download fail, and every later sync failed the same way until the user ran Clean database. Version 5.2.0 skips that record and syncs the rest. Clean database remains available as a last resort. A phone far behind catches up page by page, where v4.1.3 could run out of memory on a long backlog. Uploads get the same care. One change the server fails on (an attachment link typed without `https://`, say) can stop every upload after it; 5.2.0 sends the others anyway, sets the failing one aside as "Not accepted", with Discard or Keep, once it's clearly at fault, and while uploads keep failing, the Timeline says "The server keeps failing on uploads". The attachment dialog adds a missing `https://` and refuses a link the entry already has ("This link is already attached"), as the server would. A download the phone can't save, as when its storage is full, says so ("Sync failed (the phone's storage is full)") instead of stopping silently.

**Modern phones.** The app works with Android's current requirements: edge-to-edge screens (the app draws behind Android's status and navigation bars), predictive Back (the back gesture shows where it will go), and 16 KB memory pages (a memory layout newer phones use). Screens keep what you typed when the phone orientation is turned; and the Back button, links and shortcuts behave the same on every screen.

**Accessibility.** Every button has a TalkBack label (TalkBack is Android's screen reader), and at 200% text size the forms rearrange rather than cut words off. The tab bar's labels grow with the font size as far as all five fit; v4.1.3's stayed one size.

**Privacy and security.**

- The app no longer asks Android for an advertising ID, which v4.1.3 did through Firebase.
- Deleting an account in the app yields a prompt that explains that entries are kept, irretrievably separated from your identity, and then the entries are credited to a deleted account. The user then must type in their username to complete the account deletion process.
- Logging out ends the login on the server too, not only on the phone.
- A logout stays a logout on phones that once ran v3, which kept a second copy of the login that v4.1.3 brought back at the next start.
- Crash reports, when turned on, get a failed request's name and status code, not the server's reply or the text of one the app couldn't read, and a wrong password isn't reported as a crash.
- Android's backup of the app keeps only its settings, not the login or the data: a phone restored from a backup asks you to log in again and downloads the data again. v4.1.3 backed up everything, the login included.
- Every request names the app and its version to the server, as the iOS app does, with no phone model or ID.

## Notable changes for users

These differences from v4.1.3 are deliberate. Anything not listed should behave as it did.

| Area | v4.1.3 | 5.2.0 |
|---|---|---|
| Flags | An entry with two flags disappears from the app | The app never hides an entry because of flags. An entry can take only one flag; flagging it again is blocked. Use comments to discuss an existing flag. |
| Entries waiting to upload, on a shared phone | They went up under whoever logged in next, who could also edit or delete them | Each change waits for the account that made it, and goes up at that account's next login. Another account's waiting entries can't be edited or deleted. |
| Default favorites (new installs) | Ten: Old Faithful, Castle, Grand, Daisy, Riverside, Great Fountain, Beehive, Fountain, Aurum, Lion | Seven: the park's predicted geysers plus Beehive. Updating keeps your own favorites, except on phones still on v4.0.x (below). These settings are saved by the phone, not the GeyserTimes server. As such, they don't follow your account to another phone, though setting up a new phone from an Android backup may bring them along. |
| Starting or returning to the app | Fetches new reports when started; when returning, shows what it had until a pull to refresh. Note: Pull updates required since push updates are no longer being performed properly by the GeyserTimes server (the associated FCM API call was discontinued in 2024) | Fetches new reports when started, and when returning if the last sync is over two minutes old. Just like v.4.1.3, v.5.2.0 is designed and ready to receive push updates from the GeyserTimes server. |
| Nearby geysers | A button that suggests the geysers within 100 m | Android's "Near my precise location" button, above the Geyser field, which Google Play requires of apps built for Android 17 from January 27, 2027. Suggestions are still the geysers within 100 m, and you choose. |
| Typing a geyser's name | The submit forms suggest the geysers whose names start with what was typed, A to Z. The search bar lists every geyser, then every user, whose name contains it, A to Z | Both list the geysers whose names contain what was typed, best matches first: names that start with it, then names with a later word that does ("Mortar" finds Fan and Mortar), then the rest. Up to 50 geysers; the search bar then lists every matching user, as before. |
| Phone turned sideways | While typing, Android's full-screen keyboard: only the field and the keyboard. The date picker puts its calendar beside its header | While the keyboard is up, the form hides its title, tabs and Submit, so the field stays in view; Back brings them back. Date pickers take a typed date, since Material's calendar doesn't fit. |
| Notifications | Asks for Android's permission when first opened | The notifications setting is off by default. If the user had previously accepted notifications, updating leaves the Notifications setting on. |
| Look and Settings | Material Design | Same visual aesthetic, but with the modest use of Compose blue (#00639B) to tell the new version apart from the old; The Settings screen was reorganized and regrouped (for example, Sync & data) |
| Settings' times and names | "Last sync" in the time-zone setting's zone, unnamed. The sync interval without a timer is "Automatic" | "Last sync" in the phone's own time, with its zone ("1548:39 PDT"). That interval is "When new data is available"; only the switch is "Automatic sync". |
| Wording | "Login" and "Sign In" for the same thing; "Okay"; some labels capitalized word by word ("Note Saved") | "Log in" and "Log out"; "OK"; only a title's first word capitalized, apart from names |
| Account screen without a connection | Closes, with "Network connection is required" | Manage account stays in Settings and says so. If the screen opens but the profile doesn't load, it stays open, with the fields greyed out and a "Reload profile" button |
| Time zone | The setting has five fixed choices: Yellowstone, Iceland, New Zealand, Kamchatka and El Tatio | The setting's choices are the server's geyser fields, each with its zone, such as "Yellowstone (Mountain Time)", so a new field shows up by itself. Every screen shows and enters each eruption and note in its own geyser's zone, with a label such as "GMT+13:00" when that differs from the setting's. The Webcam's times stay in Yellowstone time. |
| A time picked just after midnight | On a form opened before midnight, a time picked after midnight was saved on the day before, 24 hours early, unless the date was changed too | On a new entry's form left open over midnight, a time picked after midnight that has already begun goes on the new day, and the date shows it. Other times, and a date you chose, stay as they are. |
| Geyser icon colors | Five Yellowstone basins in their own colors, the rest of Yellowstone brown, New Zealand blue-gray, everything else gray | A new palette. The same five Yellowstone basins have their own colors and the rest of Yellowstone shares one, cyan; each basin in New Zealand, Iceland and Chile, and Mickey Hot Springs, has its own too. Other Geyser stays gray. |
| Webcam | The stream (broken for several months up to the present), with picture-in-picture and Cast; under it, the predictions and the 10 latest eruptions | The Old Faithful stream plays again, from Pixelcaster, the Park Service's streaming service, which GeyserTimes may rebroadcast. Under it, a card for each predictable geyser the camera can see (Castle, Daisy, Grand, Old Faithful, Riverside), first the one you'd most likely miss by looking away for the coming minute. A fresh report of Beehive's Indicator, a Giant Hot Period or a Fan and Mortar Event Cycle puts that geyser's card on top for a while, as "Precursor activity since 1405". A clock on the picture gives the time it was recorded, since the video often plays 20 to 30 seconds behind. "Chat" opens chat.geysertimes.org, and "Key" shows a panorama of the view with each geyser named. Sideways, the video fills the screen, as before. No picture-in-picture or Cast, and no chat inside the app. |
| New versions | Only through Google Play's own updates, automatic or from the Play Store | Also from inside the app: when Google Play has a newer version, the app asks once, on one of the five tabs. Play downloads it while you go on, and a banner offers to restart into it; nothing installs until you tap Restart. "No thanks" holds for that version. Only a copy installed from Google Play asks, and the first question comes with the version after this one. |

Not carried over: v4.1.3's full-screen button in the webcam player. Turning a phone sideways gives a full-screen webcam view.

## Updating from v4.1.3 to 5.2.0

An update keeps your login, favorites, settings (the Summary or Timeline choice included), first date and last sync time. Entries you made offline in v4.1.3 and haven't uploaded yet stay on the phone and upload once there is signal.

- **One-time quirk:** if someone never answered v4.1.3's notification permission question, Android asks it again on the first launch, and the app closes once after the answer. It opens normally the second time.
- **If copying the old data fails:** the app copies just what v4.1.3 hadn't uploaded, which works even where the whole copy runs out of memory, and downloads the rest from the server. If that fails too, it shows an error with Try again, and tries again at the next launch; with the phone's storage full, it asks for space instead. After three failures, or three tries that closed the app, it carries on and downloads the data from the server. What v4.1.3 hadn't uploaded then stays on the phone: Sync info says so, with Try again and Discard, and each later version tries once by itself.
- **Phones still on v4.0.x:** the copy can't open their older database, so it fails as above: setup shows its error twice, then carries on with the default favorites, and Sync info offers Try again and Discard for what v4.0.x hadn't uploaded. On 2026-11-04, 56 of 1,892 phones had v4.0.4, none of which had installed v4.1.0 (July 2021) or any later version, so few are likely to update. A fix is left for a later release (app-android issue #448, part A).
- **Older phones:** Version 5.2.0 requires Android 7 (released August 2016) or later. For now, Android 5.0 and 6.0 phones may continue to run v4.1.3.

## Waiting on the server

- **Push messages don't arrive,** on either version. The server still uses Google's old way of sending them (the legacy FCM API), which Google shut down in 2024, and its push script can't reach the database. The move to the current API (FCM HTTP v1) must keep the collapse key "sync", which both versions look for. The team is working to fix this.
- **Two flags still hide an entry on the server.** 5.2.0 allows one flag per entry, but flags from the website, or from two phones before they sync, can still reach two.
