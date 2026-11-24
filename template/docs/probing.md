# Probing

How to look at a state of the window from a script, without the person's hands.

## Rules

- Probe with a second copy, never the copy the person is using: `STARTER_HOME` at a scratch folder.
- Say what was checked and what was not. A synthetic click does not reach a click handler in a background window, so a picture of the state a click leads to does not show the click works.

## A copy

```
cargo build -p starter-app
STARTER_HOME=<scratch>/home STARTER_GO=x STARTER_SHOT=<file.png> ./target/debug/Starter
```

| Variable | What it does |
|---|---|
| `STARTER_HOME` | the copy's own settings and state |
| `STARTER_APPEARANCE` | `light` or `dark`, for this launch only; never written back |
| `STARTER_LANG` | a language's code (`de`, `zh-Hans`), over the system's |
| `STARTER_SETTINGS` | open the settings on that section (`appearance`, `general`, `about`) |
| `STARTER_GO` | steps, `;` between: `fold`, `sidebar`, `work`, `menu`, `updatefound` (a newer version waits: the gear's and About's dots; nothing is fetched), `checking`, `uptodate`, `checkfailed`, `downloading`, `installfailed` (each state the Updates button comes to; nothing is fetched), `check` (the button's own press: a real check, fetched); any value keeps the window over the others |
| `STARTER_SHOT` | a picture of the window (the settings when they are open) after it is drawn, then quit; `STARTER_SHOT_FRAME=1` keeps the system's shadow and rim |

A process may take a picture of its own windows without Screen Recording access, which a terminal running a probe usually lacks.

## A press, checked

A press moves nothing but what it changes (`AGENTS.md`, A press moves nothing), and a picture of each state alone does not show that: a row that grew by one line looks finished on its own. Each state a press leads to is shot beside the state before it, at the same section, language and appearance, and compared:

```
STARTER_HOME=<scratch>/home STARTER_SETTINGS=about STARTER_GO=x STARTER_SHOT=<dir>/rest.png ./target/debug/Starter
STARTER_HOME=<scratch>/home STARTER_SETTINGS=about STARTER_GO=uptodate STARTER_SHOT=<dir>/uptodate.png ./target/debug/Starter
Rscript scripts/press-diff.R <dir>/rest.png <dir>/uptodate.png
Rscript scripts/press-diff.R --split 240 <dir>/main-rest.png <dir>/main-work.png
```

It prints, for the left column (the settings' rail, 190 wide, or the sidebar with `--split 240`) and the page apart, the box in points that holds every pixel that differs. The box must sit inside the control pressed (the Updates button is 224 by 28 at the right of its row; Start is 104 wide, its mark beside it 31), or in the rail for a dot. A box that reaches a row's or a card's edge, or anything under the control, is a press that moved the layout. The traffic lights differ when the copy was not in front as it was shot; that is not the press. The copy checks for updates as it opens, and the answer draws over a step's state, so the shots come from a home whose `config.json` says `{"app":{"check_updates":false}}`. Done in English and in German and Japanese, whose words are longest; the fixed widths were measured with the text system (`window.text_system().shape_line`) over every label each control can show, in every language.

## A corner, checked

A sharp corner is a few pixels at a box's corner and does not show in a picture of the whole window. Each rounded box's corners are cut from the probe's picture and enlarged eight times with no smoothing, in both appearances:

```
Rscript scripts/corners.R <dir>/main.png <dir>/corners.png card-bl=238,700 card-br=1058,700 box-tl=358,578
```

Each `name=x,y` is a 20-point square from its top left, in points. A corner is round in the strip: nothing of what is inside the box (a fade, a strip, a row, a line) shows square past the curve.

## Measuring

A speed-up starts from a number and ends with another. The stopwatch goes in a copy and comes out when it is done; none is kept.

- **At rest:** `top -pid <pid> -stats cpu,idlew,mem`, with the window closed and open. A product at rest is at about 0% with a few wakeups a second. What is left to find is in what it does at work: go through each path, and read what runs on a timer.
- **A frame:** the time from the top of `render` to a `canvas` painted last. A hover draws the window whole, so a frame asked for by a timer costs what a hover costs. A frame has about 16 ms.
- **A wait outside a frame:** a timer of 5 ms on the main thread that prints any gap over 30 ms. It finds a lock, a sound or a file that no frame shows.
- **A pointer with no hands:** a sweep through `window.dispatch_event` (`PlatformInput::MouseMove`, and a press and a release on each section), so the person's cursor stays where it is.
- **A wait that comes and goes:** the same thing done twice running, and done after a pause of several seconds. A system that has gone quiet takes a moment to wake.
- **A rate:** the app's CPU time and the window server's over the same seconds, doing nothing, as it was, and as it is. Frames cost the window server too, and with no baseline to take off, a saving does not show.
- **Work on the GPU:** a call that hands it work returns before the work is done, and the wait shows up under whatever comes next. Wait for it (`llama_synchronize`) before the clock stops.
- **What grows with use:** rows made up for the purpose at a size the person has and at several times it, never the person's own words.
- **Memory:** the physical footprint (`footprint <pid>`), never the resident size `top` and `ps` give. A window's drawing surfaces are IOSurfaces, which those leave out: a leak of 2 GB read there as 207 MB. `heap <pid>` counts the live objects of each Objective-C class (`GPUIView`, `CAMetalLayer`).
- **A window opened and closed** many times by a task in the copy (`remove_window`, then the window opened again), the footprint read before and after. A closed window that keeps its surfaces shows only so.
- **Launch:** the time from `main` to the first frame painted, and to each step of start-up between. A bundle's launch differs from a bare binary's (the Dock icon, the app's own defaults), so it is timed in a copy of the dev bundle with the build put in it and signed ad hoc (`codesign -s - --force --deep`).
- **Nothing lost:** what the product gives is compared with the old build's, word for word, kept beside the new one.

## The core from a terminal

```
cargo run -p starter-core -- paths | config | update
cargo run -p starter-core -- --try-update <version> <copy of Starter.app>
```
