New Features

One more thing to discuss. I'm using the position log feature a lot. To be clear, does the "Go" buttons move to the MCS or WCS that is displayed in the log? Appears to be MCS, which is what I need but want to be clear. The other thing is can you think of a way to have the position log entries persist after a page refresh or close open page event? Can you think of a better way to save these types of settings other than embedded in json in the file? If I forget to save-as I lose everything and save-as is a little inconvenient in chrome as it's burried in menu's these days. Maybe a save-as button on the page itself if that can be done. It would keep the major versioning we are currently using and add a "_xx" at the end of the file name. That's how I've been doing it now. E.g. FluidNC_Probe_Utility_v1.15.4_01.html, FluidNC_Probe_Utility_v1.15.4_02.html, etc. Or save to an external file but that sounds problemsom

---

Good questions — let me address all three clearly.

---

**Go buttons use MCS — confirmed**

Yes. `gotoPosLogAxes()` sends `G53 G0 X{mcs.x} Y{mcs.y}` — the `G53` prefix forces machine coordinate mode regardless of any active WCS offset. The WCS columns in the log are displayed as context only. This is the right behavior: MCS positions are absolute and never depend on which job origin is active, so a logged position will always get back to the same physical spot.

---

**Position log persistence + Save-As button**

Here's my read of the options:

| Option                   | What it solves                | Drawback                                     |
| ------------------------ | ----------------------------- | -------------------------------------------- |
| `localStorage`           | Survives page refresh/reopen  | Tied to browser + machine, not in the file   |
| Embed in HTML JSON block | Portable, "in the file"       | Requires Save-As to persist each session     |
| External JSON file       | Portable, not mixed into HTML | Requires file picker API or manual placement |
| **Both**                 | Best of both worlds           | Two saves instead of one                     |

**My recommendation — do both:**

1. **`localStorage` for the position log** — auto-save on every add/delete/rename, auto-load on page open. The log survives browser refresh and reopen without any action. Since Chrome treats all `file://` pages as the same origin, it persists across version files too (a bonus).

2. **"Save Copy" button in the header** — triggers a programmatic `<a download>` of the current page with an auto-incrementing `_01`, `_02` suffix based on a localStorage counter. This is genuinely better than Ctrl+S Save-As in Chrome because it pre-fills the correct filename — you just click one button and the file appears in Downloads. The saved file will have all current settings baked into the embedded JSON (same as manual Save-As) plus whatever is in the DOM at that moment.

This keeps the "portable single file" architecture intact while eliminating the two main friction points: log disappears on refresh, and Save-As is buried.

Want me to implement both in v1.15.5?
