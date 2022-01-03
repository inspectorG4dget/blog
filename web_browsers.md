# The Browser Wars

I'm not going to tout the superiority of any one browser over the rest, but here are a bunch of useful addons and other pro-tips I have found with the browsers I use.
And just because this page would be completely remiss without trashing on IE, here's a meme:

![IE is slow](http://techstuffed.com/wp-content/uploads/2016/04/995707_659653617383013_1200757877_n-1.jpg.cf_-1.jpg)

## Mozilla Firefox

Here's a list of Firefox addons I include as part of my toolchain

### TreeStyle Tab

If you're like me, you have a billion tabs open all the time. This quickly becomes very confusion for a multitude of reasons, not the least of which is the inability to track which tabs are related based on the order of their appearance in the tab bar. Yes, subsequent tabs appear immediately on the right hand side of the tab that "spawned them" (i.e. the tab from which they were opened by the "open in a new tab" feature). Still, that is the same as the "new tab" behavior, and is therefore not quite as helpful. Chrome has tried to deal with this using Tab Groups, but that's really better at solving a different problem (displaying tab names in a sufficiently wide tab, whereas the tab itself may otherwise be squished and therefore too narrow to display the tab's name)

The first thing to notice about TreeStyleTab is that it adds a sidebar to the browser window in which it displays all tabs vertically. This ensures that the number of tabs has no bearing on the tab width and therefore the ability to display each tab's name in a ubiquitous width (erm... almost; keep reading).

The next thing to notice is that this vertical space affords the ability to indent tabs. These are used to indicate "child tabs" spawned from "parent tabs".

Finally, the vertical space scrolls, which eliminates the "tabs getting squished" problem. These features combined, make this addon very helpful and vastly improve  my productivity on Firefox. It's really a shame that I can't find something similar for Chrome (if you happen to know of one, please [get in touch][contact]).

### Tab Renamer

Sometimes, the tab title is unconveniently long (even with the TreeStyleTab addon). Sometimes, this is because the webmaster used an unhelpfully long title, and other times it's because the name of the website is unhelpfully prepended to the tab title.

Even if none of these are the case, it is often helpful to be able to rename a tab. Usually this requires that `<title>` be altered in the page's sourcecode, but that is often intractable as such changes do not persist page refreshes, etc.

This is where Tab Renamer shines. It allows the user to set a title for the tab such that it persists past page refreshes and other changes. It also allows for a simple regex config to maintain page title renaming by regex matching, which persists across tabs. For example, I typically have my three main GitHub repositories open under my ZenHub tab. I use the regex feature to rename the tabs by repo title, regardless of whether I'm on issues, pull requests, or wherever else I am within the project.

## Google Chrome

This is my preferred browser for all the syncing that it offers over my devices. It also allows me to use [Chrome Remote desktop][chromeRemoteDesktop] to access my home desktop from my Chromebook

### The Marvellous Suspender

This addon allows for tabs to be "suspended", releasing their memory footprint (so if you're hurting for RAM...). Beware though, the URL /is/ magled as part of the functionalilty, which means that this messes with tab porting from desktop to mobile devices if the addon is not installed in your mobile devices.

### Session Manager

I used to use [Session Manager][sessionmanager] to remember which tab groupings belonged to suspended projects, etc. However, I don't quite see the need for this in the face of bookmark groups.

## Cross Browser

These are the addons/extensions I use across browsers

### Ghostery

[Ghostery][ghostery] is an ad blocker that has sane default configs and allows the user to override automatically detected adware on webpages.

### uBlock Origin

[uBlock][ublock] is a more granular ad blocker - really, it's a web element blocker that allows the user to block individual elements on a webpage.


[contact]: https://inspectorg4dget.com/contact
[chromeRemoteDesktop]: https://remotedesktop.google.com/
[ghostery]: https://www.ghostery.com/
[ublock]: https://ublockorigin.com/
[sessionmanager]: https://chrome.google.com/webstore/detail/session-manager/mghenlmbmjcpehccoangkdpagbcbkdpc?hl=en