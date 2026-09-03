# 1. Target-State Principles

This is the fifth and last deliverable of the dossier, and the only one that looks forward. The four
before it answer what this repository provides toward a screen-capture-and-notetaking application
and what it verifiably does not; this one sequences the work that would follow, in five phases, each
stating what it does, what it needs, what it adds, what it waits for and how a reader would know it
is finished.

Every repository locator in this document was read against branch `5.x`, commit
`0627765f01be7ea464846ea1e56bbf4e6d861bcf`; a line locator is checkable only against that revision.

Two properties of this document follow from its position at the end of the citation graph. It
originates no evidence: every finding it relies on was established in a preceding deliverable, and
it reaches that finding by naming the section that owns it. And it reaches platform evidence the
same way — where a phase depends on a mechanism absent from this repository, it cites
[platform-capture-gap-assessment.md §1](./platform-capture-gap-assessment.md),
[platform-capture-gap-assessment.md §2](./platform-capture-gap-assessment.md) or
[platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md), which is
where that mechanism and its source were assessed, rather than the platform documentation directly.
That deliverable is the dossier's single owner of external evidence, and routing through it keeps one
account of each mechanism rather than two that can drift.

**This is planning content. None of it is executed here.** No code is written, no prototype is
built, no platform is targeted, no asset is obtained. The phases are a plan a reader can evaluate,
and the five principles below are the constraints that plan is held to. Three of them exist because a
finding in the dossier makes them necessary rather than because they sound prudent, and each of those
three names the finding.

## 1.1 Cross-platform parity where feasible

A route is preferred where it presents the same application-side contract on all three targets: the
same call to start a session, the same way of naming the surface to capture, the same failure
behaviour when that surface is unavailable, and the same record stream coming out. Parity is a
property of the contract the application exposes, not of the mechanism underneath it — the mechanisms
differ per platform and cannot be made to agree.

The consequence that shapes the design is that **the contract carries an authorisation stage on
every target, and on no target is that stage satisfied by doing nothing.** The parity finding of
[platform-capture-gap-assessment.md §7](./platform-capture-gap-assessment.md) is that the three
targets differ not in whether pixels can be obtained but in who authorises the obtaining and how the
frames are then transported — and that the difference reads on two independent axes, because a
single consent axis puts a target on the wrong side of it.

On operating-system mediation the mechanisms genuinely differ, and the unit that differs is finer
than the platform and finer than the mechanism: it is the concrete route a build selects. The
mediated Linux path requires consent as a matter of contract. The modern Windows mechanism has a
picker-led initiation path, but the picker belongs to that path rather than to the mechanism, and
the route reachable from here takes the mechanism's other initiation path, addressing its target by
handle or index with no picker anywhere in the chain. The remaining assessed mechanisms — the other
Windows ones and the X11 reads — carry no operating-system step at all. So interactive authorisation
is guaranteed by contract on one platform, present on another as an initiation path the reachable
route does not take, and absent everywhere else — which is why "the platform that asks" and "the
mechanism that asks" are both the wrong unit of design. The unit is the route a build actually
instantiates, and that makes the answer a property of the build rather than of the target.

Recording visibility divides the same way, and it is a second axis rather than a second reading of
the first: a system-drawn indicator is a property of a particular route on a particular version, it
is separately gated where it exists, and the reachable Windows route requests none. So no route is
credited with an indicator it does not ask for, and none is credited with one whose appearance the
platform decides rather than the route. Both readings are
[platform-capture-gap-assessment.md §7](./platform-capture-gap-assessment.md)'s, at the width that
section states them, and §1 of the same document holds the per-route detail. On
application-level authorisation there is no asymmetry to accommodate: the requirement is
uniform across every route, including the unmediated ones, because on a route the operating system
does not mediate nothing else stands between the application and silent capture.

So the authorisation stage is present from the start, and on every route it is the application's
own. The application obtains its authorisation before it acquires a source and exposes a recording
state the user can observe for as long as capture is active — on the mediated Linux path and on the
picker-led Windows initiation path exactly as on the routes where the operating system asks nothing.
Where the platform imposes a step of its own, that step is an **additional** requirement whose
outcome may be interactive, satisfied in addition and never instead; it does not stand in for the
application's gate, and no route reaches capture with the gate skipped. The same holds for the
indicator: where a route's platform draws one, it is drawn in addition to the recording state the
application exposes and never in place of it, because a control the application does not own is one
it cannot guarantee. Both halves are requirements rather than
prudence: [functional-spec.md §1](./functional-spec.md) states that capture is authorised before it
begins, by the application, on every platform, including those on which the operating system
requires nothing; that platform mediation is no substitute for it, being absent on some targets and
a deployment property rather than a contract where it is present; and that while capture is active
the session exposes a recording state observable both in the interface and in the note stream. A
contract designed without the stage and retrofitted later either exposes the asymmetry to every
caller on every platform or hides it and misleads the caller on the platform that has it; a contract
that declares the stage and lets a platform step discharge it is worse in both directions, because
it leaves unobservable capture the normal case on the routes the operating system does not mediate,
and on the routes it does mediate it mistakes a system dialog about handing over pixels for the
session's own decision to record a person.

"Where feasible" is a real qualifier and not a hedge for its own sake. §5 carries one bridge that
the dossier can state and cannot confirm, and the principle it works under is that a divergence is
recorded with its reason rather than asserted away.

## 1.2 Minimal added dependencies

Each phase names what it adds, and where two options reach the same result the one adding least is
preferred. Two consequences run through the rest of the document.

**Route choice is conditional on what a build already has, rather than fixed here.** The two
ingestion routes by which externally acquired frames can reach the capture API are owned, with the
condition each carries, by
[current-state-capability-map.md §1](./current-state-capability-map.md), and the build flags that
decide whether either is present are inventoried in
[technical-inventory.md §5](./technical-inventory.md). Neither condition is free: the pipeline-string
route requires a pipeline terminating in an appsink under one of two accepted names
[modules/videoio/src/cap_gstreamer.cpp:1343], and the environment-mediated route requires a build
with device-opening support [modules/videoio/src/cap_ffmpeg_impl.hpp:1213] and reaches only what its
input-format lookup can resolve [modules/videoio/src/cap_ffmpeg_impl.hpp:1206-1210]. Mandating one
route in this document would add its dependency to every deployment, including deployments that
already have the other.

**A declared build default is not an availability statement.** The declared defaults of the two
relevant gates are ON for one [CMakeLists.txt:222] and platform-conditional for the other
[CMakeLists.txt:219], and neither tells a reader what a given build contains: no configure step was
run against this checkout, and a `WITH_*` request is gated further by platform visibility and
dependency detection, which is why the inventory records configured availability as unknown. Every
phase that depends on a route therefore treats availability as something observed on the build under
test, not read off a default.

The same preference decides the extraction route in §6: the in-library option is preferred, because
it adds assets rather than a native library, unless the task carries a requirement that option
cannot meet.

## 1.3 No OpenCV source modification

The application is a consumer of this library. No phase proposes changing its sources, its build
files or its defaults, and **native screen acquisition, source selection and consent sit outside
OpenCV** in the target architecture.

This is a boundary the roadmap is required to respect rather than a stylistic preference, because
placing acquisition inside the library would contradict two findings at once. No backend among the
concrete identifiers the capture API declares names a display, screen, desktop, window or monitor
[modules/videoio/include/opencv2/videoio.hpp:91-122], as
[current-state-capability-map.md §1](./current-state-capability-map.md) establishes by enumerating
the structures that would have to contain such a backend. And introducing one is not reachable from
outside the library: discovery and loading are organised around registry-known identifiers, and the
loader rejects a plugin whose declared backend identifier differs from the one it was loaded for
[modules/videoio/src/backend_plugin.cpp:400-425], so a new first-class source would need a new
identifier, a registry entry and build integration — an upstream change, assessed in
[platform-capture-gap-assessment.md §4](./platform-capture-gap-assessment.md) and out of this
roadmap's scope.

What remains inside the library is what it already offers: ingestion of a frame stream the host
acquired, the frame-processing primitives the gate and the annotations are composed from, the display
surface, and the inference surface. What sits outside it is acquisition, the choice of which surface
to acquire, the authorisation of that choice, the input-event hook, the clock, the note stream and
the persistent annotation model.

## 1.4 Explicit failure over silent fallback

A phase exit that reads "it works" while a substitution happened underneath is not an exit. Three
applications of the principle recur.

A session that cannot open the source it was asked for fails, and says which source and why; it does
not open a different one. Fallback from one ingestion route to the other is permitted only where both
resolve to the same normalised source identity, which is the policy of
[functional-spec.md §1](./functional-spec.md); absent an equivalence between the two routes' own
addressing there is no fallback, because a stream whose recorded source changed meaning midway is not
a timeline anyone can review. And a capability that the active display backend does not implement is
reported as unavailable rather than exercised silently — the case §4 turns on.

## 1.5 Independent failure domains

Capture, correlation and extraction fail independently, and the phase order reflects that: each phase
adds a domain that can fail without taking the previous one down.

A failed extraction still produces a record, which is why
[functional-spec.md §5](./functional-spec.md) carries extraction state as four values rather than a
boolean — "did not run" and "ran and found nothing" are different facts, and a consumer that cannot
tell them apart cannot re-run the failures. A correlation stream that ends abruptly is a shorter valid
stream rather than a damaged one. And a display backend that cannot offer an interaction feature
leaves capture and correlation running. Each of those is an exit criterion in the phase that owns it,
not an aspiration stated here.

# 2. Single-Platform Capture Prototype

Phase 1 of five. It exists to establish, on one platform and with one source, that frames of a screen
surface can reach the capture API under route conditions this phase observes rather than assumes,
and be displayed. Every later phase consumes its output, so it is deliberately the narrowest phase in
the roadmap: it proves the acquisition-to-ingestion path and nothing else.

**Scope:** one platform, one source, frames reaching `VideoCapture` and being displayed. The host
acquires the frames by the selected platform's own mechanism and hands them to the library through
one ingestion route; the library opens that route, retrieves frames by the two-step pull the capture
surface exposes [modules/videoio/include/opencv2/videoio.hpp:951,965], and displays them
[modules/highgui/include/opencv2/highgui.hpp:345]. Selecting the target platform is inside this
phase's scope, and so is setting the open and read timeouts explicitly (§2.3). Outside it: the change
gate, the note stream, the input-event hook, any interface beyond a preview window, and any second
platform.

**Inputs:** the selected platform's acquisition mechanism, assessed in
[platform-capture-gap-assessment.md §1](./platform-capture-gap-assessment.md),
[platform-capture-gap-assessment.md §2](./platform-capture-gap-assessment.md) and
[platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md); the chosen ingestion
route with the condition it carries, owned by
[current-state-capability-map.md §1](./current-state-capability-map.md) with its API surface
inventoried in [technical-inventory.md §1](./technical-inventory.md) and the flags that decide its
availability in [technical-inventory.md §5](./technical-inventory.md); the source-addressing
encoding and the limitation on it, from
[platform-capture-gap-assessment.md §4](./platform-capture-gap-assessment.md); and the pipeline
requirements this phase is built to satisfy — session lifecycle, explicitly named source selection,
conditional route selection, and the rate-and-resolution knobs — from
[functional-spec.md §1](./functional-spec.md).

One of those route conditions is a constraint on the host process's own startup order rather than on
its configuration, and it is an input here because the route this phase selects can carry it. Where
the X11 route through the media framework's screen-source element is the selected route, Xlib
threading initialisation — the `XInitThreads()` call — happens **before** the library, the display
backend, or any worker thread the application starts. Setting the plugin-load-time environment
variable `GST_XINITTHREADS=1` instead is **not equivalent** to it: that variable causes the call only
when the plugin is loaded, which can be later than the first thread and so too late. The condition
is established, with the element's own account of it, in
[platform-capture-gap-assessment.md §2](./platform-capture-gap-assessment.md).

**Added dependencies:** the selected platform's acquisition mechanism, which is host code outside
this library and carries whatever the assessed mechanism itself requires; and, on the pipeline-string
route, the media-framework plugin that provides the screen-source element, which is a genuine added
package rather than a build flag. On the environment-mediated route the addition is a build whose
media backend includes device-opening support [modules/videoio/src/cap_ffmpeg_impl.hpp:1213] together
with the demuxer that acquisition mechanism is exposed as. Nothing is added to this library, and no
part of its source is changed.

Naming a dependency is not the same as accepting one, and every addition above is a native component
that runs inside the capture process. So each is recorded with five facts before this phase can
exit, and the record is of what was resolved rather than of what was requested: the exact resolved
version, not a range and not "current"; the official distribution channel it came from, named
specifically enough that a second person could obtain the same artefact from the same place; the
integrity evidence that the artefact obtained is the artefact that channel publishes, verified
against the publisher rather than against the copy just downloaded; the component's support state at
that version, meaning whether the version is still receiving security fixes and when that ends; and
the result of a published-advisory review at that version, with the fixed version named where an
advisory applies and the action taken recorded. A component that is out of support, or that carries
an applicable advisory with no available fixed version, **blocks this phase** rather than being
carried into it with a note — the phase's own route-availability finding is what would otherwise
launder an unmaintained plugin into the prototype's evidence. This applies to every addition equally,
including the ones the deployment installs as packages and the ones the host platform supplies,
because the process does not distinguish them at load time. It applies to this library itself on the
same terms: the version under test is recorded, and where plugins are enabled the discovery controls
that decide which native library the process loads are set explicitly, per
[technical-inventory.md §5](./technical-inventory.md) and
[current-state-capability-map.md §6](./current-state-capability-map.md).

**Five facts about a named component are not the whole of the dependency, so the record is a complete
resolved bill of materials rather than a list of the components a phase happens to name.** Every
operational dependency any phase introduces or relies on is enumerated in it, direct **and**
transitive, to the leaves of the graph, and each entry is recorded at the version actually resolved on
the build under test rather than the version requested — a request is a range or a package name, and a
range resolves differently on a different host and on a different day, so a bill of materials of
requested versions describes no build that exists. The published-advisory review runs against that
whole graph and not against the named components alone.

The transitive half is the load-bearing half, and the reason is what the components do rather than how
many there are. A screen-source element or a portal client is a thin piece of code over the libraries
that do the work — the decoders, parsers, transport, compression and image libraries beneath it — and
those are what read bytes the session did not author, from a display server, a portal, a media
transport or an asset file. So the layer that carries the memory-safety and parsing risk is usually not
the layer a phase names, and a graph recorded only to its first level can be entirely clear while the
component actually exposed to those bytes is unpatched. A graph recorded to its leaves is also what
makes an advisory actionable in the other direction: when one names a library rather than a plugin,
the question "is that library in this deployment, at what version, underneath what" has an answer
already written down instead of an investigation that starts after the advisory.

**Every entry in that bill of materials has a named accountable maintenance owner, and the
responsibility does not end when the phase does.** Recording an advisory review at an exit is a
point-in-time act: it says what was known on the day of the exit, and an advisory published the
following week has no addressee unless one was named. So each entry — direct and transitive, installed
or host-supplied — carries the person or the specifically named team accountable for advisory response,
patching and end-of-life migration for that component, together with how they are reached and what
they have accepted; a role in the abstract is not an owner, because "the operator" and "whoever
deploys it" name nobody who can be told. The responsibility runs for as long as the component is in
the deployment, past this phase's exit date and past every later phase's, which is what makes the
record a living one rather than an artefact of the exit. Where a component is one the target platform
supplies and its patches come from that platform's own maintainers, the named owner is whoever in the
deployment is accountable for tracking that upstream and acting on what it publishes — upstream
maintainers are not an addressee for this deployment's decision to keep running an unpatched host. An
entry with no named owner **blocks this phase in exactly the way an out-of-support component does**,
and for the same reason: both are components nobody is going to fix. The maximum time a deployment
allows between an applicable advisory and a patched deployment is a product decision, and the request
supplies none, so where no value is supplied that part of the record is **recorded as blocked pending
that decision** rather than written against an interval this dossier would have had to invent — the
owner, the graph and the blocking outcomes stand and are evaluable without it.

Those five facts, the resolved bill of materials and the named owner are one rule, and this is the
dossier's single statement of it. Every later phase references it as **the dependency rule of §2**
rather than restating it — Phase 2 (§3) for the input-hook facility, Phase 3 (§4) for its considered
answer that it adds no component, Phase 4 (§5) for its per-target additions, and Phase 5 (§6) for its
assets and its external engine — because a rule restated five times is a rule with a clause dropped in
one of them.

**Sequencing:** none; this phase is first. It has no predecessor, and nothing in it waits on another
phase. The one ordering constraint it does carry is internal to the process it builds: where the X11
route above is the selected one, Xlib threading initialisation precedes library initialisation,
display-backend creation and the start of the first worker thread, which places it at the top of the
application's own startup path where no later step can reorder it.

**Exit criteria:**

- The selected platform is recorded together with the evidence for its selection: which mechanism,
  which ingestion route, the observation that the route's own condition held in the build under test,
  and — where the route chosen is a candidate carrying open items — the settlement of every one of
  those items that is material to the selected target and route. An item material to what this phase
  selected is this phase's to settle, not Phase 4's: the eligibility gate of §2.1 refuses a target
  while such an item is unsettled, so a target that reaches this exit has none left open. Only open
  items belonging to a target this phase considered and did not select pass to Phase 4 (§5)
  unsettled, because those are the targets Phase 4 takes up. Targets considered and not selected are
  recorded with the evidence that excluded them, so the exclusion is re-derivable rather than
  assumed.
- Frames from an explicitly named source arrive and are displayed. Named means the application states
  which surface it opened rather than accepting whatever an auto-detection sentinel resolves to.
- A request for an unavailable source fails explicitly, naming the source and the reason, and no
  other source is opened in its place.
- The route's build conditions are recorded as observed rather than as declared defaults, including
  the result of the backend-availability probe the public registry exposes
  [modules/videoio/include/opencv2/videoio/registry.hpp:45].
- The selected route's initialisation order is recorded as it actually occurred, not as intended. On
  the X11 route through that screen-source element the record states the order in which Xlib
  threading initialisation, library initialisation, display-backend creation and the first worker
  thread's start took place, states that the explicit `XInitThreads()` call was made in the
  application's own startup path, and states that no equivalence was assumed between that call and
  the plugin-load-time `GST_XINITTHREADS` variable — the condition being the one
  [platform-capture-gap-assessment.md §2](./platform-capture-gap-assessment.md) establishes. On any
  other selected route the record names the route and the initialisation order that route's own
  conditions require, so the exit is evaluable whichever route this phase chose.
- The open and read timeouts are set explicitly through the parameter vector the open overloads accept
  [modules/videoio/include/opencv2/videoio.hpp:877,901], and the values used are recorded.
- The authorisation step runs before any capture begins and the recording state is observable while
  capture is active, demonstrated on the selected platform whether or not that platform's mechanism
  carries an operating-system step of its own, per
  [functional-spec.md §1](./functional-spec.md) and the principle of §1.1.
- The consent and recording-visibility behaviour **of the concrete route this phase selected** is
  recorded from observation on the target, not carried over from the mechanism family that route
  fronts. The record answers three questions for that route as instantiated: whether any
  operating-system selection step appeared at all, and if so what selected the surface; whether the
  platform drew a recording indicator, and on what version and under which capability or consent it
  did or did not; and what the route requested in each case, kept distinct from what the platform
  then did, since a request to suppress an indicator is not a suppression and a route that requests
  none may still be given one. Where the route addresses its target by handle or index there is no
  selection step to observe, and the exit is met by recording that absence rather than by finding
  one — the per-route positions being
  [platform-capture-gap-assessment.md §1](./platform-capture-gap-assessment.md)'s and their parity
  reading [platform-capture-gap-assessment.md §7](./platform-capture-gap-assessment.md)'s. This exit
  does not license skipping the one above: the application's own authorisation and its own
  observable recording state are required on this route whatever this record says, and this record
  exists so that a later phase inherits the route's real behaviour rather than the family's.
- Every added dependency is recorded under the dependency rule of the added-dependencies section
  above, in full: the five facts per component — exact resolved version, official channel, integrity
  evidence verified against the publisher, support state at that version, and advisory review with
  the action taken — the complete resolved bill of materials, direct and transitive, at the versions
  the build under test actually resolved, with the advisory review run over that whole graph rather
  than over the named components, and a named accountable maintenance owner for every entry in it
  whose responsibility continues past this exit. An out-of-support component, one carrying an
  applicable advisory with no fixed version available, or an entry with no named owner leaves this
  exit **unmet and blocks the phase**, and the record states which component and which finding did
  it.
- The captured source is a platform object rather than a free-form string, obtained by whichever of
  the two selection shapes the chosen mechanism uses: enumerated, where the mechanism exposes
  addressable targets and host platform code can list them before choosing; or granted, where the
  identity does not exist until the platform mediates the choice and returns it, in which case the
  returned object is normalised into the session's source identity after the grant. Which shape
  applies follows from the mechanism selected, per
  [platform-capture-gap-assessment.md §1](./platform-capture-gap-assessment.md) and
  [platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md). Under either shape
  the route argument is assembled from the application's allowlist of elements and properties plus
  that platform-issued identifier, with any value carrying the route's own delimiters or
  metacharacters rejected rather than escaped, and with nothing a user typed carried into it, per
  [functional-spec.md §1](./functional-spec.md). The route argument is what decides which components
  a session instantiates — the pipeline string is parsed by the media framework itself where it is
  neither a valid URI nor an existing file [modules/videoio/src/cap_gstreamer.cpp:1432,1438], and the
  environment-mediated route's options are parsed as delimited pairs
  [modules/videoio/src/cap_ffmpeg_impl.hpp:1197] — so this exit is about the construction of that
  argument and not about validating one after the fact.
- Acceptance thresholds for capture frame rate and resolution are taken from user-supplied values
  where any exist. None were supplied with this request, so this exit is **recorded as blocked pending
  that product decision** rather than written against a figure this dossier would have had to invent.

## 2.1 The target platform is selected, not assumed

The phase's first activity is to select the platform, and the selection is an activity with an
evidence requirement rather than a decision this document makes. Nothing in the dossier ranks the
three targets, and ranking them here would substitute a preference for a finding.

The selection is gated on two conditions holding together for the same target. The platform must
have an acquisition mechanism the application can use in its intended deployment posture — attended
or unattended, whole-monitor or single-window, with or without an operating-system step of its own
on top of the application's own authorisation — which is what
[platform-capture-gap-assessment.md §1](./platform-capture-gap-assessment.md),
[platform-capture-gap-assessment.md §2](./platform-capture-gap-assessment.md) and
[platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md) assess per platform.
And an ingestion route must carry that mechanism's output into the library, with the route's own
condition satisfied in the build under test, which is what
[technical-inventory.md §5](./technical-inventory.md) supplies the flags for and what
[platform-capture-gap-assessment.md §4](./platform-capture-gap-assessment.md) records as
route-by-route bridging.

Neither condition disqualifies a target in advance on the evidence this dossier holds, and the
eligibility gate is written against evidence rather than against an inherited verdict. On the
mechanism side, every target has at least one assessed mechanism whose output an ingestion route can
carry: [platform-capture-gap-assessment.md §1](./platform-capture-gap-assessment.md) records a route
for each of the Windows mechanisms it assesses — two of them fronted by a single pipeline element,
and the oldest reachable through either route;
[platform-capture-gap-assessment.md §2](./platform-capture-gap-assessment.md) records a route for
the X11 read; and [platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md)
names a concrete candidate chain for the mediated path, together with the items a build has to
settle before that candidate is a route. So the mechanism side is satisfiable on all three targets,
and no target is ineligible for lack of a mechanism that can reach the library.

The discriminator is therefore the evidence this phase itself collects, in two parts. **Which route
condition holds in the build under test:** each route's condition is owned by
[current-state-capability-map.md §1](./current-state-capability-map.md), with the flags that decide
availability inventoried in [technical-inventory.md §5](./technical-inventory.md); this phase neither
restates those conditions nor relaxes them. What it adds is observation of the route it intends to
use in the build it is using — that the route's condition was satisfied there, that the component
performing the acquisition was present in that installation, and that the route opened a source
rather than merely being compiled in. **And whether the open items a candidate route carries have
been settled for the target under consideration:** for the mediated path those are the items
[platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md) enumerates —
ownership and lifetime of the transport descriptor across the hand-off, negotiation to a format the
sink accepts, and behaviour when the session ends underneath the pipeline.

Which phase owns those items follows from selection and not from convenience. Every item material to
the target and route **this phase selects** is settled in this phase; an item material to the
selected route cannot be deferred, because the route is what this phase exists to prove. Items
belonging to a target this phase considered and did not select stay open and pass to Phase 4 (§5.2),
where those targets and their items are that phase's declared work.

A target is eligible for this phase when both conditions are observed to hold for it in the build
under test, and it is not eligible while a route condition fails there or a material open item is
unsettled — a statement about the evidence in hand rather than about the platform, and one this
phase can change by gathering more. The gate and the exit therefore say the same thing from two
directions: a target carrying an unsettled material open item is not selected, and a target that is
selected reaches the exit with none of its own left open. Recording which targets were eligible on
that basis, with the evidence for each and the reason for each exclusion, is part of this exit, and
it is also the input Phase 4 (§5) picks up.

## 2.2 Route availability is a property of the build under test

The two routes are not interchangeable, and this phase records which one it used and why. Which of
them is the stronger, and the condition each carries, are settled by
[current-state-capability-map.md §1](./current-state-capability-map.md) and inventoried in
[technical-inventory.md §1](./technical-inventory.md); this phase takes that comparison as given
rather than re-deriving it, because a second statement of a condition is where the condition gets
weakened. What this phase owes is the observation: which route it used, that route's condition
observed to hold in the build under test, and the evidence for both.

Neither route is discoverable through the registry, which is the finding
[platform-capture-gap-assessment.md §4](./platform-capture-gap-assessment.md) states as a delta: the
application cannot ask the library whether a screen source is available and must carry that knowledge
itself. The practical consequence for this phase is that its record of route availability is an
artefact the application produces, not a query result — the availability probe
[modules/videoio/include/opencv2/videoio/registry.hpp:45] answers whether the backend is present in
the build, which is a necessary condition and not a sufficient one.

## 2.3 Timeouts belong to this phase, not to a later hardening pass

Two timeout identifiers are declared, both documented open-only and both applicable to two backends
only [modules/videoio/include/opencv2/videoio.hpp:187-188]. Those lines establish that the
identifiers exist and what they apply to; the values live in each backend independently, and both
define a 30-second default [modules/videoio/src/cap_ffmpeg_impl.hpp:261-262] and
[modules/videoio/src/cap_gstreamer.cpp:83-84].

For an interactive capture session a stall of that length on open or on read is a user-visible
failure, so setting both explicitly is in this phase's scope rather than deferred: a prototype that
inherits the defaults will appear to work and will hang in front of a user the first time a source
goes away. The narrow applicability is part of the requirement, as
[functional-spec.md §1](./functional-spec.md) specifies — on a backend outside those two the
identifiers buy nothing, and the session's own supervision is what bounds a stalled open. Recording
which of the two situations the prototype is in is part of recording the route's build conditions.

## 2.4 Thresholds this phase cannot invent

This is the discipline that separates the roadmap from a wish list, and it applies again in §6.

No numeric latency, throughput, availability or coverage threshold is committed anywhere in this
repository, and no service-level agreement is defined, so no exit criterion in this document
attributes a performance number to OpenCV. Frame rate and resolution enter the dossier as tradeoffs
expressed against the property knobs that exist, which is how
[functional-spec.md §1](./functional-spec.md) states them and how
[technical-inventory.md §1](./technical-inventory.md) inventories them.

That absence does not excuse an exit criterion nobody can evaluate. Acceptance thresholds for capture
frame rate and resolution are product decisions, and the request supplies none. The exit therefore
takes user-supplied values where they exist, and where none exist it is **recorded as blocked pending
that product decision** — a state a reader can act on, unlike a threshold invented to fill the gap.
Every other exit in this phase is evaluable as written, so the blocked item is one line of the exit
rather than the whole of it.

# 3. Event Correlation & Timeline

Phase 2 of five. Frames alone are a recording; frames ordered against the user's input events are
the artefact this application exists to produce. This phase builds that ordering and the stream that
records it.

**Scope:** the correlation contract of [functional-spec.md §2](./functional-spec.md) — one clock per
session, a monotonic time and a sequence number on every record, and a merge rule that is a total
order — and the note stream of [functional-spec.md §5](./functional-spec.md), which is the taxonomy,
the common field set, the screenshot addressing and the checked append the contract is recorded
through. The frame-admission gate is implemented here too, because the stream records admitted frames
and the gate is what decides admission. So does the representation of annotation revision records in
that stream, which is a property of the format rather than of the interface that produces them — the
lifecycle that produces them is Phase 3's work (§4). Outside this phase's scope: any interface beyond
the preview Phase 1 established, and any extraction of content from the frames.

**Inputs:** the session clock, the sequence-allocation point and the merge rule, from
[functional-spec.md §2](./functional-spec.md); the record taxonomy, the durability mechanism and the
failure model that mechanism is declared against, from
[functional-spec.md §5](./functional-spec.md); the OS input-capture component, which is host code and
whose boundary is assessed in
[platform-capture-gap-assessment.md §5](./platform-capture-gap-assessment.md); and what the capture
side supplies toward time and delivery, which is assessed in
[current-state-capability-map.md §1](./current-state-capability-map.md) and inventoried in
[technical-inventory.md §1](./technical-inventory.md).

**Added dependencies:** the platform input-hook facility, which is host code outside this library.
The delivery of pointer and keyboard events by the display module is window-scoped, which is what
[platform-capture-gap-assessment.md §5](./platform-capture-gap-assessment.md) records as the gap, so
a session-wide event stream is acquired by the host and not by this library. Nothing is added on the
library side: the clock, the merge, the note stream and the admission gate are all application code
written above the capture object (§3.2), not packages to be brought in.

That one addition is a dependency in exactly the sense §2 means, and it is the addition most easily
let through, because it does not arrive looking like a package: it is a facility the host platform
exposes, reached through the platform's own interface rather than installed from a catalogue. It is
also the addition with the least margin for a gap. It runs inside the capture process, and what it
observes is every keystroke and every pointer event the session sees — including the ones the
redaction of [functional-spec.md §2](./functional-spec.md) removes from the stream, since redaction
acts on an event the facility has already delivered. A component in that position is the last one that
should escape a dependency gate. So it is recorded under **the dependency rule of §2** in full, and
that rule's blocking outcomes apply here unchanged: an out-of-support facility, an applicable advisory
with no fixed version available, or an entry in its graph with no named accountable maintenance owner
blocks this phase. Two of the rule's clauses do most of the work on this particular component. Its
version and support state are properties of the target platform rather than of anything the deployment
installs, so the record names them per target and names who in the deployment tracks that platform's
advisories and acts on them — "the platform supplies it" identifies no addressee. And whatever the
facility pulls in behind its interface belongs in the same bill of materials, at the versions resolved
on the target, because the code that will handle raw input events is not necessarily the interface the
application calls.

**Sequencing:** after Phase 1, which supplies the frame stream. The gate compares a frame against the
previous retained frame, so it has nothing to compare until frames arrive.

**Exit criteria:**

- Every record kind carries a monotonic time value from the session's single clock and a sequence
  number unique within the session, each assigned at the moment
  [functional-spec.md §2](./functional-spec.md) fixes for it and not at the other: the time value is
  stamped where the record is acquired, so that the timeline records the user's actions rather than
  the pipeline's latency, and the sequence number is assigned by the session's single writer as it
  serialises the record. This phase records that no acquiring thread issues a sequence number of its
  own, because two allocation points would leave the equal-time rule below to a race between
  threads.
- The merge order is demonstrated as the ascending lexicographic comparison of the pair that
  [functional-spec.md §2](./functional-spec.md) defines — the monotonic time value, then the
  sequence number — on records of every kind the format defines rather than on frames and input
  events alone. Nothing else enters the comparison: not the civil-time field, not a record's kind.
  The load-bearing consequence is the one a reviewer reads the timeline for, and this phase
  demonstrates it as a property of how the writer allocates rather than of how the reader compares.
  Within a group of records sharing one monotonic time value the session's single writer serialises,
  and therefore numbers, input records before frame records; at equal time the input record's
  sequence number is the lower one, so the two-key comparison places the event that caused a change
  before the frame showing it with no exception inside the comparison itself. The exit records both
  halves — that the reader compares the pair and nothing else, and that the writer's allocation order
  at equal time is the one just stated.
- An interrupted session leaves a readable stream **to the failure mode the deployment declared and to
  no wider one** (§3.3). The declared mode — process-level interruption, system-level interruption, or
  neither — is named in the phase's result and recorded in the session's own `start` configuration, and
  the demonstration is an interruption of the kind that mode describes: the process killed for the
  process-level declaration, and power loss or a stopped kernel for the system-level one. A stream that
  survived a killed process establishes the process-level mode and says nothing about the other, so the
  exit is met by the demonstration matching the declaration rather than by the stronger-sounding claim.
  In every mode the result of every append, flush and synchronisation is checked, a failure is reported
  and the record treated as not written, and nothing further is appended to a stream after a failed or
  short append. Where system-level durability is the declared mode, the exit additionally requires the
  per-record durable synchronisation **and** the containing directory's metadata synchronisation of
  §3.3, both demonstrated, since the first without the second leaves a record on the device that
  nothing can reach by name. Media or filesystem failure is outside every mode and outside this exit:
  what answers it is the second copy the artefact inventory below carries, not the write path. Where the
  host supports neither mode the exit is met by recording the narrowed guarantee — complete lines parse
  — and stating that narrowing rather than the stronger one, all on the terms
  [functional-spec.md §5](./functional-spec.md) sets.
- The change gate is implemented to the score contract of
  [functional-spec.md §3](./functional-spec.md), with the per-pixel sensitivity, the admission
  threshold and the quiet-frame count that closes a segment configurable, and with the score's
  definition, range, whole-frame domain, first-frame rule and inclusive comparison fixed.
- Segment boundaries appear in the stream as records rather than being left for a consumer to
  re-derive.
- The annotation revision records of [functional-spec.md §5](./functional-spec.md) are represented in
  the stream to the extent that contract settles them: a stable annotation identifier under the
  character set that section specifies, the three revisions it defines, the field binding a revision
  to the attached frame's sequence number, current state as the fold of one identifier's revisions in
  ascending sequence order, undo as that fold stopped one revision short, and reopen as a
  deterministic reconstruction from the stream alone. Which of the format's record kinds carries a
  revision is **recorded as open** by that same section, and this phase adopts neither candidate
  carrier, so this exit is **blocked pending that specification decision**: the fold, the ordering
  and the determinism are demonstrable ahead of it and are demonstrated here, while the on-disk
  carrier cannot be fixed until the decision is taken.
- Where the backend supplies a presentation timestamp it is recorded on frame records as
  supplementary media metadata, and it is used for no ordering decision (§3.1).
- Key redaction is in force before any stream is retained: key content is excluded where the
  platform marks the field secure or password-bearing and redacted by default otherwise, the record
  keeping the event and losing only its content, and pointer records carry only the fields the merge
  and the later aggregation consume — per [functional-spec.md §2](./functional-spec.md). This is an
  exit of this phase and not of a later hardening pass, because a character once written into the
  stream cannot be unwritten by any policy applied afterwards.
- The platform input-hook facility, and everything its use brings in behind that interface, is
  recorded under **the dependency rule of §2** — the five facts per component, the complete resolved
  bill of materials direct and transitive at the versions resolved on the target with the advisory
  review run over that whole graph, and a named accountable maintenance owner for every entry whose
  responsibility continues past this exit. An out-of-support component, one carrying an applicable
  advisory with no fixed version available, or an entry with no named owner leaves this exit **unmet
  and blocks the phase**, and the record states which component and which finding did it. Phase 1's
  record does not discharge this exit: the components are different ones, and the component this
  phase adds is the one that sees every keystroke — the last addition in the plan that should reach a
  deployment ungated.
- The note stream is written under the storage protections and the identifier rules of
  [functional-spec.md §5](./functional-spec.md): the stream and the image directory created
  accessible only to the owning account at the moment of creation, under a configured storage root
  that is never derived from record content, with `session_id` and `annotation_id` restricted to the
  specified character set and the session directory created exclusively so a collision fails the
  start rather than joining an existing session, the image extension drawn from the application's
  allowlist, every artefact created and opened relative to an already-open handle on that root
  following no link or reparse point, and deletion under the retention policy leaving each record and its
  reference in place, so that a reference which no longer resolves reads as that deletion rather
  than as a damaged stream. Containment is demonstrated by how the files are opened and not by a
  pathname check performed before opening them: the check on the resolved path stays as a cheap
  early rejection, and the exit is the descriptor-relative behaviour, because a check and a
  subsequent open on the same name are two operations with a window between them.
- The integrity metadata written with each retained image is demonstrated to do what
  [functional-spec.md §5](./functional-spec.md) claims for it and is recorded as doing no more: a
  one-sided change — a truncated or corrupted image, or an image replaced under an unchanged record —
  is detected, and a change made to the image and the record together is not, because the digest is
  unkeyed and lives in the same writable artefact. Where the deployment's threat model includes an
  actor who can write to that artefact, this exit is met only with authentication rooted outside it,
  and where it does not, the exit is met by recording that the pair is a consistency check rather
  than evidence of origin. An exit claiming authenticity from the digest alone is not met.
- **Every artefact this phase causes to exist is inventoried, owned and bounded, not just the two it
  writes directly.** The exit is the complete inventory of
  [functional-spec.md §5](./functional-spec.md) instantiated for this deployment: each entry present
  with a named accountable owner, access restriction applied at creation by whoever creates it, a
  retention deadline recorded against it, and the erasure limits written down as they actually are
  rather than as a deletion feature would imply. This phase is where the estate stops being two
  files: the correlated timeline it builds adds an authorisation audit log and, where the deployment
  keeps one, an operational log of its own removals; its writer adds temporary and staged files, and any encoded session video, exported copy,
  reopen working copy, backup or snapshot belongs to the same inventory from the moment it can
  exist. Two entries need their owner named rather than assumed, and are the ones an inventory
  usually omits: a backup or filesystem snapshot the application neither created nor can reach, and
  any external component that receives recognised text or derived actions. For both, the exit is met
  by naming the holder and what that holder has accepted, and it is not met by leaving the entry
  unowned. Where a copy leaves the storage root the exit requires transit protection and a
  destination that accepted the same obligations, since access restriction does not travel with a
  file that is copied out from under it.
- **Production use of what this phase builds is a blocked state until those controls are in force,
  and "available" does not satisfy this exit.** The distinction is the whole point of the criterion:
  a configuration option that exists and is switched off protects nothing, and a phase that exits on
  the option's existence has verified the option rather than the protection. So the exit is the
  observed state of each control on the deployment that will run — every inventory entry owned, its
  access applied at creation, its retention deadline set, encryption at rest actually in force where
  the threat model requires it with the key material held by a named owner under a stated lifecycle,
  authenticated integrity present on the same condition, and both audit logs written and themselves
  inventoried. A missing control leaves this exit **unmet and blocks production use**; it does not
  convert into a risk accepted on the phase's behalf by nobody. All of it is stated as requirements
  with identifiers in [functional-spec.md §5](./functional-spec.md), so each is checkable
  individually against that section rather than as a single judgement about readiness. This phase
  cannot itself decide the retention periods, which are a product decision this request does not
  supply; where none is supplied that part of the exit is **recorded as blocked pending that
  decision**, and the rest of the criterion stands and is evaluable without it.

## 3.1 Four adjacent properties, and why the clock is the application's

The capture surface declares four timing-adjacent properties, and three of them look like the fourth
at a glance. A media-timeline position [modules/videoio/include/opencv2/videoio.hpp:133-134]; a
presentation timestamp of the most recently read frame, read-only, one backend only, and expressed in
the frame-rate time base [modules/videoio/include/opencv2/videoio.hpp:205]; a civil-time instant at
which the stream was opened, read-only and one backend only
[modules/videoio/include/opencv2/videoio.hpp:189]; and a presentation value supplied *to* the writer
when encapsulating externally encoded video
[modules/videoio/include/opencv2/videoio.hpp:230]. The precise negative
[current-state-capability-map.md §1](./current-state-capability-map.md) reaches, and
[platform-capture-gap-assessment.md §5](./platform-capture-gap-assessment.md) carries as a delta, is
that none of them is a backend-independent per-frame host-clock acquisition instant.

The presentation timestamp is genuine per-frame metadata and is recorded as such where available. It
is never substituted for the session clock: it is a media timestamp in a media time base, it is
backend-specific, and a timeline built on it would change meaning with the route the frames arrived
by. The facility the application reads to obtain a monotonic instant is not provided by the in-scope
modules, so the clock is the application's own, which is what makes the single-clock rule of
[functional-spec.md §2](./functional-spec.md) a design decision this phase implements rather than a
library behaviour it configures.

## 3.2 The gate sits above the capture object

Nothing in the capture surface hands the application a frame unasked. Retrieval is a two-step pull
[modules/videoio/include/opencv2/videoio.hpp:951,965], and the plugin binary interface carries the
same shape with no push or event-driven entry point
[modules/videoio/src/plugin_capture_api.hpp:92,103]. The one readiness API
[modules/videoio/include/opencv2/videoio.hpp:1035-1053] requires every capture in the call to share
a backend and dispatches only to the V4L backend, raising `StsNotImplemented` for any other
[modules/videoio/src/cap.cpp:630,652].

So the negative this phase is built against is a scoped one, and
[platform-capture-gap-assessment.md §4](./platform-capture-gap-assessment.md) is the single site that
owns it: there is no readiness or frame-available signal at the capture plugin binary interface, and
none for a non-V4L backend, the V4L exception being real and reaching neither of the two ingestion
routes a screen source arrives by. A screen route is therefore polled, and the consequence is
architectural rather than inconvenient: the session's own retrieval loop drives the timing, and the
gate that decides which frames are retained sits above the capture object in application code. The
three operators a frame-to-frame gate is composed from are not provided by the in-scope modules, as
[current-state-capability-map.md §2](./current-state-capability-map.md) records on the capability
side and [functional-spec.md §3](./functional-spec.md) on the requirement side; what the in-scope
modules do supply is the preprocessing and the interpretation of a difference image, inventoried in
[technical-inventory.md §2](./technical-inventory.md). The gate is therefore built in this phase, to
the score contract, and it is not configured into existence.

## 3.3 What "readable after interruption" means as an exit

The exit is stated against a mechanism **and against a named failure mode**, because neither the
format nor the word "flushed" bounds loss on its own. A line-delimited stream guarantees only that a
complete line parses independently; a half-written record is invalid regardless of how its fields are
ordered. And "interruption" is three different events with three different answers, which is the
distinction [functional-spec.md §5](./functional-spec.md) draws and the one this exit is written
against:

- **Process-level interruption** — the application is killed, it crashes, or the operating system
  stops it. A record whose append the operating system has accepted survives this, because the file
  it went into does not belong to the process. What does not survive is anything the writer was still
  holding in a buffer of its own, which is why the mechanism is one checked append per record, each
  record serialised in full and newline-terminated, with no record held in a buffer of the
  application's own between records: a writer that flushes every few records loses whichever ones were
  pending, and that count is a property of a buffer size rather than of any contract.
- **System-level interruption** — the machine loses power, the kernel stops, the host is destroyed
  mid-session. An append the operating system accepted may not have reached the storage device, so
  survival here does **not** follow from the append returning. It requires the platform's durable-sync
  primitive to have completed for the file, and the containing directory's metadata to have been
  synchronised as well, both per record. The second half is the one a deployment forgets and it is not
  optional: a synchronised file under an unsynchronised directory entry survives as content nothing
  can find, and every artefact this phase writes is reached by name. The cost is a synchronisation per
  record and it is the deployment's to accept; a deployment unwilling to pay it declares the
  process-level mode instead of performing the cheaper mechanism and describing the stronger one.
- **Media and filesystem failure** — the device fails, or the filesystem is corrupted. **No promise is
  made here at all**, by this phase or by the specification it implements, because no ordering of
  writes answers a device that has stopped working. What answers it is a second copy, which is an
  entry in the artefact inventory this phase's exits already require, carrying the same owner, access
  restriction, retention deadline and transit obligations as every other entry — a backup rather than
  a durability mechanism, and never described as one.

Which of the first two modes is in force is a **deployment declaration and not a property this
roadmap can assert**, since the mechanism differs per host and the two promises differ by the whole of
power loss. The declaration is recorded in the session's own `start` configuration, so a stream says
which mode produced it and a reader of the stream is not left to infer the promise from the word
"flushed". No guarantee wider than the declared mode is stated to a user or to a consumer, and the
declared mode is what this phase's exit is evaluated against — not the strongest-sounding of the
three.

Checking is a requirement in every mode, including the cheapest one. The result of every append, every
flush and every synchronisation is examined; a failure is reported through the session's own error path
and the record is treated as not written, because an unchecked write is a record silently absent from a
stream in which absence is undetectable — one line fewer looks exactly like one event fewer. A short
append carries its own rule: it leaves a partial line, a partial line is discardable only as the
stream's last line, and so the writer appends nothing further to that stream after a failed or short
append and the session's recording path ends there and reports why, rather than continuing over a file
whose next line would turn a discardable tail into a corrupt middle.

Where the host supports neither mode — an append whose result the platform cannot report, or storage
that acknowledges writes it has not taken — the guarantee narrows to what the format supports on its
own, which is that complete lines parse, and the phase records that narrowed guarantee rather than
claiming either of the stronger two.

# 4. UI Shell

Phase 3 of five. Capture and correlation now run; this phase gives a user something to watch, mark up
and control. It is the phase whose exits are most conditional, because a public declaration in the
display module does not imply that the active backend implements it.

**Scope:** preview, controls, annotation rendering, the annotation lifecycle and the timeline
scrubber. Preview is the continuous presentation of captured frames; controls are session start and
stop and the gate's sensitivity and threshold; annotation rendering is the composition of marks,
boxes and text onto the frame before it is displayed. The annotation lifecycle is the five operations
[functional-spec.md §6](./functional-spec.md) assigns to the application — create, update, delete,
undo and reopen — driven over the revision records
[functional-spec.md §5](./functional-spec.md) specifies and Phase 2 represents in the stream; this
phase owns them because they are interface behaviour, and no later phase would pick them up. The
scrubber is the frame-index control that same interface section requires. Outside this phase's scope:
any second platform, and the format-level representation of a revision record, which is Phase 2's
(§3).

**Inputs:** the display and interaction verdicts with the backend condition each carries, assessed in
[current-state-capability-map.md §3](./current-state-capability-map.md) and inventoried in
[technical-inventory.md §3](./technical-inventory.md); the interface requirements and their verdicts,
from [functional-spec.md §6](./functional-spec.md); the revision and fold contract the annotation
lifecycle is driven against — the three revisions, the binding to the attached frame, current state
as the fold of one identifier's revisions in ascending sequence order, and the determinism that
makes a reopened stream reconstruct the same state — from
[functional-spec.md §5](./functional-spec.md); and the finding that display and interaction
parity is backend-conditional rather than platform-conditional, from
[platform-capture-gap-assessment.md §7](./platform-capture-gap-assessment.md).

**Added dependencies:** none beyond the display backend already present in the build Phase 1 used —
unless a control this phase requires exists on one backend only, which would add one. The button
control is declared for a single toolkit [modules/highgui/include/opencv2/highgui.hpp:808-810] whose
gate carries a declared default of OFF [CMakeLists.txt:299], so a control surface that depends on it
adds that toolkit to every deployment. A control surface built from trackbars and keyboard input adds
nothing, and is preferred for that reason under §1.2.

"Nothing" is stated here as a considered answer rather than left as a silence, because a phase that
says nothing about dependencies is indistinguishable from a phase that did not look. The considered
answer is that this phase introduces no component of its own: the display backend it uses, the toolkit
libraries that backend sits on and this library itself are already entries in the bill of materials
Phase 1 (§2) recorded, at the versions that build resolved. They do not leave **the dependency rule of
§2** by being inherited rather than added — the named accountable maintenance owner of each entry
carries the responsibility past Phase 1's exit date, so an advisory published against the display
toolkit while this phase is running already has an addressee, which is the whole point of the owner
being named rather than the review being filed. What would change the answer is the one branch above:
a control surface that depends on the single-toolkit button control adds that toolkit and its own
resolved graph, direct and transitive, as a genuine new addition, and it enters under the same rule
with its own five facts, its own place in the bill of materials and its own named owner — blocking
this phase on the same terms if any of the three is missing. Carrying a component's whole lifecycle,
and not only its installation, is a further reason §1.2 prefers the surface that adds none.

**Sequencing:** after Phase 2, which supplies both the frames the shell previews and the timeline it
presents. The annotation-lifecycle exits additionally consume the representation Phase 2 establishes,
so they close only once the carrier decision that
[functional-spec.md §5](./functional-spec.md) **records as open** is taken. The operations, the fold
and the determinism are specified here so that the exit is evaluable the moment it is, and until then
those exits are **blocked pending that specification decision** while every other exit in this phase
stands on its own.

**Exit criteria:**

- The active backend is identified at runtime through the public probe
  [modules/highgui/include/opencv2/highgui.hpp:261] and named in the phase's result. An exit that
  does not name the backend has not been evaluated, it has been assumed.
- Preview presents **captured** frames continuously, not admitted ones (§4.2).
- The event pump runs: one of the event-fetch calls is invoked periodically and at least one window
  exists and is active, which the header states as the condition for any event being handled at all
  [modules/highgui/include/opencv2/highgui.hpp:282-287].
- Interaction features degrade explicitly where the active backend does not implement them: the
  feature is reported unavailable rather than registered and silently ignored.
- Any exit asserting pointer input or trackbars names the backend it was verified against. The
  framebuffer backend accepts a mouse-callback registration and a trackbar creation, logs that
  neither is supported, and does nothing with either
  [modules/highgui/src/window_framebuffer.cpp:322-324,327-333], so an exit verified there says
  nothing about an exit on another backend, and the reverse.
- Annotation rendering is demonstrated as composition into the frame before display, and the
  persistent annotation state it implies is recorded as belonging to the application rather than to
  this module.
- Each of the five annotation operations is demonstrated end to end over the revision records of
  [functional-spec.md §5](./functional-spec.md): create writes a revision carrying the annotation's
  identifier with its initial geometry and text; update writes one carrying the identifier and the
  changed fields only; delete writes a tombstone revision carrying the identifier and no payload;
  undo is demonstrated as the fold of that identifier's revisions stopped one revision short of the
  latest, with no revision removed from the stream; and reopen is demonstrated as a deterministic
  reconstruction of every annotation's current state from the stream alone, with no side file and no
  in-memory state carried across the restart. The demonstration binds each revision to the frame it
  is attached to and folds one annotation's revisions in ascending sequence order — the fold order
  [functional-spec.md §5](./functional-spec.md) fixes for a single annotation, which is not the
  timeline merge order Phase 2 established, that being over records of every kind on the pair of
  monotonic time and sequence. These exits inherit Phase 2's open carrier item, as **Sequencing**
  above records.
- The timeline scrubber is demonstrated as a trackbar over the index of the session's admitted
  frames, created and read through the trackbar surface with its bounds and position both settable
  and readable [modules/highgui/include/opencv2/highgui.hpp:517,558,571,532,545], and the backend it
  was verified against is named — the same backend-conditional discipline every other interaction
  exit here carries, because a backend that logs a trackbar creation as unsupported and does nothing
  with it [modules/highgui/src/window_framebuffer.cpp:327-333] yields no scrubber at all. Where the
  named backend does not implement trackbars the exit is met by reporting the scrubber unavailable
  rather than by presenting a control that does not move.
- This phase's dependency answer is recorded as an answer rather than omitted as a nil return: either
  that no component was added and the entries this phase relies on remain under **the dependency rule
  of §2** with their bill-of-materials versions still those of the build under test and their named
  maintenance owners still current, **or** — where the single-toolkit control was required — that
  toolkit and its resolved graph recorded in full under that rule. An out-of-support component, one
  carrying an applicable advisory with no fixed version available, or an entry with no named owner
  leaves this exit **unmet and blocks the phase** exactly as it would in the phase that first
  installed it, since an inherited component is not a reviewed one.
- The recording indicator is verified to be visible for as long as capture is active, and the
  backend it was verified against is named — the same discipline as every other interaction exit
  here. It is composed into the presented frame rather than carried by the window title, which
  [functional-spec.md §6](./functional-spec.md) requires because a title conveys nothing on a
  backend that logs it as unsupported and discards it
  [modules/highgui/src/window_framebuffer.cpp:319], and it is a distinct element rather than an
  inference from the preview updating.

## 4.1 Every display and interaction exit is backend-conditional

Backend-compatible registry membership is a fixed list built into the module
[modules/highgui/src/registry.impl.hpp:27-66]: the GTK family, a framebuffer backend, a Windows
backend compiled only on that platform, and one further entry that is present in the source but
disabled behind a compile-time guard [modules/highgui/src/registry.impl.hpp:51] — what the guard
shows is work started or considered and left incomplete, and the code records no reason. That list
is what can be selected through the module's internal backend interface, and it is not the whole of
backend identity: the public probe returns the active backend-compatible backend's name where one is
present and otherwise names a legacy compile-time built-in
[modules/highgui/src/window.cpp:1096-1121], the set the installed header enumerates for it
[modules/highgui/include/opencv2/highgui.hpp:256-261], which is the distinction §5.3 works from.
Which implementation is active in a given build is not decidable from this document either way,
which is why the probe result — rather than membership in that list — is the exit criterion rather
than a premise.

The condition is not a formality. The public surface declares windowing, resizing, titles, trackbars,
pointer callbacks and region selection, and one backend implements several of them as a logged
refusal, as [current-state-capability-map.md §3](./current-state-capability-map.md) establishes and
[technical-inventory.md §3](./technical-inventory.md) records per entry point. An application that
reads the declaration as a guarantee will register a pointer callback, receive no events, and have no
error to show the user. Determining the backend at runtime and degrading explicitly is therefore the
same principle as §1.4 applied to the display surface.

## 4.2 Preview presents captured frames, not admitted ones

The distinction decides whether the shell feels broken. The admission gate of
[functional-spec.md §3](./functional-spec.md) governs what is retained in the note stream and what
screenshots are kept; applying it to the preview would freeze the displayed image for the whole of
every quiet period, which is precisely when a user is most likely to check that capture is still
running. So the preview consumes the captured stream, and the gate consumes it separately for
retention.

Frame-by-frame display at video rates is what the module documents itself as supporting: the display
call's own note pairs it with a short event-pump wait for exactly this use
[modules/highgui/include/opencv2/highgui.hpp:332-333]. Any throttling of the preview below the
capture rate is a separate requirement with its own justification, and this phase does not introduce
one implicitly by reusing the gate.

## 4.3 What this phase decides, and what it does not

The verdict [functional-spec.md §6](./functional-spec.md) reaches on multi-panel layout, docking,
menus and text entry is that they are absent from the module, and the frame for saying so is the
module's own account of itself as a facility for trying functionality quickly and visualising results
[modules/highgui/include/opencv2/highgui.hpp:55-66]. A production notetaking interface needs a
general interface toolkit, and this phase's shell is a preview surface with simple controls rather
than a substitute for one.

Which toolkit is a decision this roadmap requires and does not take. No interface toolkit or
component library is specified for this application, and naming one here would be an invention rather
than a requirement — the same failure as asserting a capability no source establishes. So the
decision sits **inside** this roadmap as a product decision that is **blocked pending a user-supplied
choice**, and what it gates is stated rather than left implicit: until it is taken, the production
interface — multi-panel layout, docking, menus and text entry, which
[functional-spec.md §6](./functional-spec.md) finds absent from the module — cannot be built, and no
phase in this roadmap can exit on one. Placing it outside the plan would have left the target
application unreachable by any phase, which is the failure this entry exists to prevent.

This phase's own exit is unaffected by that block, and that is the point of separating them: it is
met by a preview-and-simple-controls shell that works on the backend it names, with the features that
backend does not support reported as unavailable. The blocked decision is recorded against the
production interface, not against the shell.

# 5. Cross-Platform Capture Parity

Phase 4 of five. One platform captures, correlates and displays. This phase asks what it takes for
the other two to do the same behind an unchanged application-side contract, and it is the phase most
likely to end with a recorded divergence rather than a clean parity claim.

**Scope:** the remaining two platforms behind the same application-side contract the prototype
established — the same session lifecycle, the same explicitly named source selection, the same
failure behaviour, the same record stream. The contract is held constant and the mechanisms behind it
are allowed to differ. Outside this phase's scope: any change to the contract itself, which would
invalidate Phase 1's and Phase 2's exits, and any extraction of frame content.

**Inputs:** all three platform assessments and the parity summary —
[platform-capture-gap-assessment.md §1](./platform-capture-gap-assessment.md),
[platform-capture-gap-assessment.md §2](./platform-capture-gap-assessment.md),
[platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md) and
[platform-capture-gap-assessment.md §7](./platform-capture-gap-assessment.md) — together with the
capture surface and its routes from
[current-state-capability-map.md §1](./current-state-capability-map.md), the display surface from
[current-state-capability-map.md §3](./current-state-capability-map.md), their inventories in
[technical-inventory.md §1](./technical-inventory.md) and
[technical-inventory.md §3](./technical-inventory.md), and the pipeline contract being held constant
from [functional-spec.md §1](./functional-spec.md).

**Added dependencies:** each additional platform's acquisition mechanism, with whatever that
mechanism itself requires, plus the ingestion route's own dependency where the second and third
platforms need a different one from the first. On the consent-mediated target — the Wayland path
assessed in [platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md) — the
additions are three: the portal client that negotiates the session and holds it open, the
media-transport client that carries the frames, and the media-framework plugin providing the source
element the candidate chain of §5.2 names, the first two host-side and the third a package the
deployment installs. That chain is what this phase validates rather than assumes (§5.2). The display
side of that target is a separate matter from acquisition and adds nothing here: a build in which
the option is requested and the dependencies resolve selects the Wayland built-in display backend
[modules/highgui/CMakeLists.txt:55-57], which is absent from backend-compatible registry membership
[modules/highgui/src/registry.impl.hpp:27-66] and so not selectable through that path, while the
public probe still reports `WAYLAND` from its compile-time branch
[modules/highgui/src/window.cpp:1116-1117], and its own build gate carries a declared default of OFF
[CMakeLists.txt:235]. So this phase gates display parity on the probe result plus per-operation
capability, never on registry membership (§5.3). Where the X11 target is one of the two added here,
it brings with it the initialisation-order condition Phase 1 (§2) records for that route and
[platform-capture-gap-assessment.md §2](./platform-capture-gap-assessment.md) establishes, unchanged
and not restated.

Every addition here is recorded under **the dependency rule of §2** in full — its five facts per
component, its complete resolved bill of materials direct and transitive with the advisory review run
over that whole graph, and its named accountable maintenance owner per entry continuing past this
phase's exit — and an out-of-support component, one carrying an applicable advisory with no fixed
version available, or an entry with no named owner blocks this phase in the same way. The rule is not
restated because it does not change; what changes is how much it has to cover.
This phase adds the most components of any in the plan and the three on the consent-mediated target
are the ones a dependency record most easily loses: the portal client and the media-transport client
are host-side, so their version and support state are properties of the target distribution rather
than of anything the deployment installs, and the source-element plugin is a package whose own
version is distinct from the media framework's. Each is recorded separately at the version actually
resolved on each target, because the answer differs per target and a single row covering "the media
framework" would hide that. The bill of materials is per target for the same reason: each target
resolves its own graph, and an entry's version, support state and owner can each differ across them.
Where a target's own components are what fail the support or advisory check, the phase records the
divergence against that target and does not assert parity for it, which is the same outcome the
criteria below already require for a blocked bridge.

**Sequencing:** after Phase 3, so that the contract being held constant already exists and has been
exercised end to end on one platform. Attempting parity before there is something to be at parity
with produces three implementations and no contract.

**Exit criteria:**

- Each additional platform reaches the same application-side contract, **or** its divergence is
  recorded with the reason and with what the divergence costs a caller.
- The authorisation path is exercised on the consent-mediated platform on **both** paths — a first
  run, and a restored session where persistence was requested and granted — and the behaviour
  observed on each is recorded per target environment rather than predicted. The two are not assumed
  to differ: whether either presents a prompt is decided by the mediating backend and the
  compositor's policy, a persistence grant is optional so a restore token exists only where it was
  requested and granted, a token that does exist is single-use and replaced on each successful
  restore, and a stored session that cannot be restored is prompted for as it would be without one,
  all of which [platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md) states.
- **Where a restore token exists, its handling is exercised as sensitive capability state and not
  merely as a string that makes the second run quieter.** That is what the token is: the one artefact
  able to carry a screen-capture grant from one run into the next, so an actor who reads it inherits
  the quieter path the user granted to this application. The exit is therefore the five owners
  [platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md) assigns, each
  demonstrated rather than asserted. The stored value is held in the platform's protected
  application storage scoped to the account that owns the session, with no second copy anywhere and
  none of it in an environment variable, a command line or a configuration file that travels with
  the output. It appears in no note record and in no log line, diagnostic, error message or crash
  report — checked by searching the artefacts a run actually produced for the token's value, which
  is the only form of this check that establishes anything, and the artefacts searched are the whole
  inventory Phase 2 (§3) established rather than the note stream alone. Replacement after a
  successful restore is atomic, demonstrated by interrupting between the old value's use and the new
  value's arrival and observing that what remains is one intact value and not a torn one, with an
  unreadable stored value discarded and the session authorised afresh rather than retried against
  it. Erasure is demonstrated on revocation, at logout or uninstall, and on a failed policy check.
  And the application's own authorisation gate is exercised both with a token present and with none,
  showing the gate's outcome unchanged by the token's presence — a possessed token authorising a
  session by itself is this exit failing, not passing, since it would mean the application had
  delegated its own decision to a value on disk. Where the target grants no token, the exit is met
  by recording that no token was granted, which is the same evidence in the negative and not an
  exemption.
- The consent and recording-visibility behaviour of **each additional platform's selected concrete
  route** is recorded from observation on that target, on the terms Phase 1 (§2) sets for its own
  route: what selection step appeared if any, whether the platform drew a recording indicator and
  under what version and capability, and what the route requested as distinct from what the platform
  did. It is not inherited from the mechanism family, and it is not inferred from another target's
  answer. This is where the parity claim is at most risk of being made on the wrong evidence: two
  targets can front the same mechanism family and reach it by routes that differ on both axes, so
  the comparison [platform-capture-gap-assessment.md §7](./platform-capture-gap-assessment.md) draws
  is only sound over routes as instantiated, and the per-route positions it rests on are
  [platform-capture-gap-assessment.md §1](./platform-capture-gap-assessment.md)'s. A target whose
  route supplies neither a selection step nor an indicator is not a divergence to record against the
  contract, because the contract never depended on either; what would be a divergence is the
  application's own authorisation or recording state differing across targets, and that is the exit
  below.
- Where validating the candidate chain of §5.2 establishes a blocker on one of its named items, the
  phase **records that as unresolved**, names the blocking item, and does not assert parity for that
  platform.
- The deployment posture each platform actually supports is recorded — attended or unattended, and
  whole-monitor or window-scoped — rather than assumed uniform across the three.
- Display and interaction parity is reported separately from capture parity: for each target, the
  framework the public probe names at runtime
  [modules/highgui/include/opencv2/highgui.hpp:261] and, for the backend it names, which of the
  operations the shell uses that backend implements. The report is per backend rather than per
  platform, and it is never inferred from registry membership (§5.3).
- The application-level authorisation path and the observable recording state are exercised on
  **every** target, including the targets whose mechanisms carry no operating-system step, since
  those are precisely where the requirement of [functional-spec.md §1](./functional-spec.md) is the
  only thing standing between a session and capture the user cannot detect (§1.1). A target on which
  the operating system asks nothing is not thereby exempt from this exit; it is the target the exit
  exists for.

## 5.1 What parity is, and what it is not

The parity substance, so that this phase has a criterion rather than an aspiration: the platforms
differ **not in whether pixels can be obtained but in who authorises the obtaining and how the frames
are transported**, which is the comparison
[platform-capture-gap-assessment.md §7](./platform-capture-gap-assessment.md) draws across the three
assessments, on the two axes §1.1 works from. The X11 path admits an unmediated read
[platform-capture-gap-assessment.md §2](./platform-capture-gap-assessment.md). The Windows path
offers both postures at once, and which one a build gets is decided by the route rather than by the
platform: one mechanism has a picker-led initiation path whose outcome is interactive, the same
mechanism has a second initiation path that addresses a target directly and asks nothing, the other
mechanisms carry no operating-system step at all, and the modern ones are bound to a graphics device
[platform-capture-gap-assessment.md §1](./platform-capture-gap-assessment.md). The route reachable
from here is not the picker-led one, so on the evidence this dossier has, the Windows target's
practical posture is the unasked one. The mediated Linux
path negotiates a session with a separate service, under a policy the application does not own, and
delivers frames over a separate transport
[platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md).

What follows for the contract is the shape §1.1 already committed to, and it is three things rather
than one. A platform authorisation step — additional to the application's own, and potentially
interactive in its outcome — guaranteed by contract on one target, available on a second through an
initiation path the reachable route does not take, and absent from the third; which means the
contract must accommodate the step without any target being relied on to provide it, and means the
count is a property of the routes a build selects and is therefore an output of this phase rather
than an input to it. A recording indicator divides the same way and is likewise not relied on: where
a platform draws one it is additional, and the reachable Windows route requests none. A session
lifetime that only one target
imposes, which is what is genuinely distinctive about the mediated path — a source selection
callable once per session, an optional persistence grant, a single-use restore token where one is
returned, and a transport addressed separately from the session. And the application's own
authorisation and observable recording state on all three without exception: on the targets whose
routes the operating system does not mediate nothing else would reveal that a session is running,
and on the mediated routes the platform's step is satisfied in addition rather than instead. That is
why this phase's exit is about the contract holding rather than about the mechanisms converging, and
why the posture each platform supports is recorded rather than presumed — an application designed
around unattended operation on one target may require a user interaction on another, and that is a
contract-visible difference even when the frames are identical.

The mechanisms themselves, and the sources establishing them, are not restated here. They belong to
the three platform sections, which is the single-ownership rule this document works under, and a
second account of a mechanism is exactly where its condition gets dropped.

## 5.2 The consent-mediated bridge is a candidate this phase validates

This phase carries one dependency whose end-to-end path the dossier can state and cannot confirm,
and writing it as either settled or impossible would misreport it in opposite directions.
[platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md) names a concrete
candidate chain and sources each of its links separately: the mediated session yields a transport
descriptor and a monotonic serial identifying the granted stream, the media framework's source
element for that transport accepts exactly those two values as properties, and the library accepts a
manual pipeline whose terminating element is an appsink under one of two accepted names
[modules/videoio/src/cap_gstreamer.cpp:1343], searching the parsed pipeline for an element so named
[modules/videoio/src/cap_gstreamer.cpp:1502] and failing where there is none
[modules/videoio/src/cap_gstreamer.cpp:1534]. The chain is
`pipewiresrc fd=<fd> target-object=<serial> ! <conversion> ! appsink name=appsink0`, with the session
opened and held by the application and both values obtained from it.

So this phase's work on that platform is the validation of a named candidate rather than a search for
a mechanism, and what it validates is enumerated rather than left as "the bridge". Six items: three
are the ones §3 records as unsettled — the descriptor, the negotiation and the failure behaviour —
and three are conditions the chain carries on its face:

- **Session lifetime and its owner.** The session belongs to the application; the library has no
  session concept to own it with. The phase records which component opens the session, holds it and
  closes it, and how that lifetime is ordered against the capture object's own.
- **Transfer of the descriptor.** The descriptor belongs to the session and is handed to an element
  constructed inside the library's pipeline. Who duplicates it, who closes it, and in what order
  relative to the pipeline's teardown is documented by neither side of that boundary, so the phase
  establishes it by observation rather than by reading.
- **Target identity by serial.** The granted stream is addressed by its monotonic serial rather than
  by a reusable identifier, and the phase records that the element was targeted that way and that the
  identity is re-established correctly after a restore.
- **Caps negotiation.** The sink path has to negotiate a format it accepts, and the source may
  negotiate buffers carrying their own memory type rather than plain system-memory video — which is
  why the chain shows an explicit conversion stage. The phase records the conversion that resolved
  the negotiation and the format the sink actually received.
- **The sink-naming condition.** The pipeline terminates in an appsink named as the library requires
  [modules/videoio/src/cap_gstreamer.cpp:1343]; a chain that satisfies everything else and fails this
  is rejected at open with no frames delivered
  [modules/videoio/src/cap_gstreamer.cpp:1534].
- **Failure behaviour.** What the chain does when the session is revoked, or when a persistence
  grant's token is replaced, needs a defined answer, because the session can end without the pipeline
  being told. The phase defines that behaviour and demonstrates it, under the explicit-failure
  principle of §1.4.

The exit admits three outcomes: validation succeeds and the contract holds; validation succeeds and
the contract holds with a recorded divergence; or validation establishes a blocker on one of the six
items, which the phase **records as unresolved**, naming the blocking item and leaving the platform
outside the parity claim. The third outcome is a legitimate result of this phase, not a failure of
it, and Phase 1's evidence-based platform selection (§2.1) is what keeps the roadmap from having
depended on it earlier.

## 5.3 Display parity is a separate question from capture parity

The two are independent and are reported separately. Capture parity is about acquisition mechanisms
and ingestion routes; display parity is about which display backend is active on each platform and
which operations that backend implements — the condition §4 established, which is a property of the
backend rather than of the platform.

Two levels of backend identity have to be kept apart for that report to be correct, and
[platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md) is where the
distinction is drawn. Backend-compatible registry membership is the built-in list
[modules/highgui/src/registry.impl.hpp:27-66], and absence from it bounds what can be selected
through the registry path and nothing more. Compile-time built-in identity is the other level, and
the public probe reports it: the probe consults the backend-compatible backend first and falls
through to a compile-time branch when there is none
[modules/highgui/src/window.cpp:1096-1121], its own documentation naming the frameworks it can
return that way [modules/highgui/include/opencv2/highgui.hpp:258-261]. A target whose display
backend is outside the membership list can therefore still be the framework the probe names at
runtime. Collapsing the two levels produces a wrong verdict in either direction — a target reported
as having no display surface when the probe names one, or a probe result read as a guarantee that
every declared operation works.

So this phase builds the display report from what the probe returns on each target
[modules/highgui/include/opencv2/highgui.hpp:261] and, for the backend it names, which of the
operations the shell uses that backend actually implements. The second half remains as §4 found it: a
public declaration does not imply uniform support, and one backend accepts a mouse-callback
registration and a trackbar creation and does nothing with either beyond logging that neither is
supported
[modules/highgui/src/window_framebuffer.cpp:322-324,327-333], refusing window titles the same way
[modules/highgui/src/window_framebuffer.cpp:319].

The two questions can diverge on the same target in either direction: a verified capture path
alongside a probe-named backend that previews but carries no pointer input, so the session captures
there and its interaction features are reported unavailable; or a backend implementing every control
the shell needs while that target's capture route is the unresolved one of §5.2. Reporting one number
for "platform support" would hide both cases, which is why the exit asks for the probe result and the
per-operation support of the backend it names.

# 6. Text/Action Extraction Layer

Phase 5 of five, and last because it consumes both of the streams the earlier phases produce. It adds
meaning to the artefact: what text was on the screen, and what the user did.

**Scope:** text extraction and action extraction, kept separate. They share this phase because both
enrich existing records, and they share nothing else: their inputs, their failure modes, their
dependencies and their exits are distinct, and a single combined exit would let one pass on the
strength of the other. Outside this phase's scope: learned visual classification of actions (§6.3),
and any change to the record taxonomy, which
[functional-spec.md §5](./functional-spec.md) fixes rather than leaving configurable.

**Inputs:** for text — the detector and recogniser assets and the character vocabulary, with the
provenance and licence of each recorded, whose concrete dependency identity is named in
[platform-capture-gap-assessment.md §6](./platform-capture-gap-assessment.md) and whose capability
baseline is assessed in
[current-state-capability-map.md §4](./current-state-capability-map.md) and
[current-state-capability-map.md §5](./current-state-capability-map.md) and inventoried in
[technical-inventory.md §4](./technical-inventory.md). For actions — the correlated input-event
stream from Phase 2, which is what actually knows a click or a keystroke occurred, and the enumerated
aggregation taxonomy with its rule set and its confidence semantics, from
[functional-spec.md §4](./functional-spec.md).

Provenance and licence are the wrong two facts to stop at, and stopping there is the specific way an
asset input goes wrong. Provenance says where an artefact is claimed to be from and licence says what
may be done with it; neither says that the bytes in hand are the bytes that source published, and
neither says anything about what happens when they are parsed. A model file, a configuration file, a
character vocabulary, an external engine's language data and a label set are all inputs read by
native code before any recognition happens, so each is admitted on six facts rather than two. Its
origin is the publisher's own distribution channel, named specifically, and not a mirror, a
model-sharing account of unclear ownership or a link from a paper. Its version is exact and pinned to
an immutable identifier, since an asset name that resolves to different bytes over time is not
pinned. Its integrity is established by a digest or signature obtained from the publisher through a
path independent of the artefact itself, verified before first use and re-verified whenever the
artefact changes — a digest published beside the file it describes, or recomputed from the copy just
downloaded, establishes only that a download completed. Its declared format, expected size and the
resources its parse may consume are stated in advance, so an asset that is not what it claims to be
is refused on a stated bound rather than discovered during a parse. Its parsing runs at the least
privilege that works, and where the format's parser is large or the asset's origin is anything short
of first-party, that parse is isolated from the session's own data and from the note stream. And each
asset is entered in the bill of materials alongside the components of §2, at the exact version
resolved, reviewed for published advisories against the asset, against the runtime that loads it and
against that runtime's own resolved graph — the parse happens below the interface the application
calls, which is why the graph rather than the named runtime is what the review covers. An asset
failing any of the six is not admitted as an input to this phase, which is a stronger statement than
it appears: it means the phase cannot begin on an asset that merely looks right, and the licence
record — which is a real requirement and stays — is not what admits it.

**Added dependencies:** on the in-library route, the model assets and the vocabulary — artefacts
rather than a library, which is why §1.2 prefers this route. On the alternative route, an external
recognition engine and its language data, which adds a native library with its own licensing,
packaging and build integration. That engine is a native component and is recorded under **the
dependency rule of §2** on the same terms as any other — its five facts, its complete resolved bill
of materials direct and transitive with the advisory review run over that whole graph, and its named
accountable maintenance owner continuing past this phase's exit — in addition to the asset rules its
language data carries as an asset. Action extraction on the two routes this phase implements adds
nothing: it consumes the event stream Phase 2 already produces, so its dependency answer is that
considered nothing rather than an unexamined one.

**Sequencing:** last. Text extraction needs admitted frames and their changed regions; action
extraction needs the correlated timeline; and both write into a record taxonomy Phase 2 established.

**Exit criteria:**

- **Text exit.** The chosen assets are integrated, the vocabulary is supplied to the recogniser, and
  the result is evaluated against a labelled sample of the actual screen content the application
  targets rather than a general document corpus. The accuracy threshold is taken from a user-supplied
  value where one exists; none was supplied with this request, so this exit is **recorded as blocked
  pending that product decision**.
- **Action exit.** The aggregation taxonomy is enumerated and closed, each entry's rule is defined,
  confidence is recorded as the two-valued rule-satisfaction indicator rather than a probability, and
  every derived action is written where [functional-spec.md §4](./functional-spec.md) places it: as an
  entry in the actions array inside the extraction object of the frame record the action anchors to,
  selected by the deterministic anchor rule that section fixes, with no new record kind introduced.
  Each entry is verified to carry all of its specified fields — its own identifier, its type drawn
  from the closed taxonomy, the monotonic start and end values taken from its contributing events
  rather than read live, the sequence numbers of the input records that contributed to it, the anchor
  frame's own sequence number written explicitly so the entry resolves when read out of context, the
  normalised source identity, the confidence indicator, and the geometry where the rule produced one
  — since those linkage fields are what makes an action re-derivable from the stream instead of an
  assertion in it. An entry missing any of them fails this exit. The array is populated independently
  of text extraction, and the mixed case where one half succeeded and the other did not is carried by
  the extraction status the format already defines rather than by any addition to the taxonomy.
  Action-coverage thresholds are the same kind of product
  decision, so where none is supplied that part of the exit is **recorded as blocked pending that
  decision** while the taxonomy and rule definitions remain evaluable as written.
- Every asset carries a recorded provenance and licence, since model licensing varies independently
  of this library's.
- **Every asset is admitted on the six facts the inputs section above requires, and the exit is the
  verification rather than the record of it.** Named channel, exact immutable version, integrity
  verified against the publisher through a path independent of the artefact, declared format with
  stated size and resource bounds, least-privilege or isolated parsing, and advisory review of the
  asset and of the runtime that loads it. The evidence is what was actually done: the digest or
  signature compared and against which published value, the bound that a malformed input was
  observed to be refused on, and the privilege the parse actually ran at. An asset whose bytes cannot
  be tied to its publisher, or whose version cannot be pinned to something immutable, **is rejected
  and this exit is unmet** — the phase does not proceed with it under a note, because a recognition
  model is native-code input on every frame the session admits and a wrong one is wrong on all of
  them. The same terms apply to the external-engine route without change: its language data is an
  asset by this definition, and the engine itself is additionally a native component under **the
  dependency rule of §2**, which brings its complete resolved dependency graph and a named accountable
  maintenance owner with it, so that route is admitted on both sets rather than on either — and an
  engine that is out of support, unpatched against an applicable advisory, or unowned blocks this
  phase on the rule's own terms.
- An asset that changes after admission is treated as a new asset and re-verified, and a build that
  cannot show which exact asset version produced a recorded evaluation result **does not meet the
  text exit** — an accuracy figure that cannot be attributed to specific bytes is not attributable
  to anything, which is the same reason the evaluation is against a labelled sample of the actual
  screen content rather than a general corpus.
- Extraction failure is demonstrated to be independent of capture: a frame whose extraction failed is
  still an admitted frame and still produces a record, with the four-state extraction status
  [functional-spec.md §5](./functional-spec.md) specifies.
- Extraction output is subject to the same storage protections and retention rules as the records it
  enriches — owner-only permissions applied at creation, the configured storage root, the restricted
  identifiers created and opened relative to the open root handle, and deletion under the retention
  policy leaving each record and its reference in place, so that a reference which no longer
  resolves reads as that deletion — per [functional-spec.md §5](./functional-spec.md). Recognised
  text is a transcript of whatever was on the screen, so an extraction payload written outside those
  rules would reopen on the derived data exactly what Phase 2 closed on the source records. It is worse than a reopening,
  in fact: a screenshot has to be looked at, while a transcript of the same pixels is searchable, so
  the derived artefact is the more exposing of the two and is the one an inventory built around
  images tends to miss.
- **The artefact inventory Phase 2 (§3) established is extended to everything this phase adds, and
  the deployment gate is re-evaluated over the extended set.** This phase creates artefacts the
  earlier inventory could not have listed, and each takes an owner, an access restriction applied at
  creation, a retention deadline and its erasure limits on the terms
  [functional-spec.md §5](./functional-spec.md) sets: the extraction payloads themselves, any
  intermediate crop or preprocessed region written to disk on the way to a model, whatever the
  runtime caches of a loaded asset, and any store belonging to an external component. The external
  case is the one that needs naming rather than assuming, because it is where the estate leaves the
  machine: an external recognition engine or service that receives frames or regions is a processor
  of this material, recorded as such before it is used, bounded to the purpose the session was
  authorised for, and inventoried with its holder named — and where such a component retains
  anything, the phase records what it retains and for how long, or records that it could not
  establish either, which is itself the finding. Passing pixels to something outside the session and
  treating the result as though only the result existed is the failure this exit prevents. Since the
  gate of Phase 2 is stated over the whole inventory rather than over a fixed list, extending the
  inventory re-opens it: **production use stays blocked until the controls are in force for the new
  entries too**, on the same terms and with "available" no more sufficient here than there.

## 6.1 The condition every text verdict carries, and where confidence stops

The inference surface this phase builds on is present and the assets are not, which is the verdict
[current-state-capability-map.md §4](./current-state-capability-map.md) reaches. Two consequences are
this phase's to handle.

Recognition is conditional on what the caller supplies. A recogniser needs a character vocabulary
[modules/dnn/include/opencv2/dnn/dnn.hpp:1880] and a decode type
[modules/dnn/include/opencv2/dnn/dnn.hpp:1857], and neither has a default this tree provides; a text
verdict stated without that condition is a verdict about a build nobody has.

Confidence is not uniform across the two halves of the surface. Detection yields geometry with a
detector confidence [modules/dnn/include/opencv2/dnn/dnn.hpp:1910], while recognition returns strings
and no per-string confidence, as
[functional-spec.md §4](./functional-spec.md) records. So a per-string confidence is the
application's to produce — through a custom decoder or an external engine — and this phase either
builds it or records its absence in the extraction payload. Implying that the surface supplies it
would misstate what a consumer of the note stream can filter on.

## 6.2 Two action routes here, and one deferred

Directly observed actions come from the correlated event stream: a click is a click because the
stream says so, which is the primary route and needs no model. Deterministic aggregation composes
higher-level actions from those events plus segment geometry and duration, against the closed taxonomy
and rules [functional-spec.md §4](./functional-spec.md) enumerates, with confidence as a record of
which clauses of a rule fired. Both are implemented in this phase, and both are inspectable — a
property worth keeping, because a reviewer of a notetaking artefact needs to know why the timeline
says what it says.

## 6.3 Learned visual classification is a later option, with its prerequisites

It is out of this phase's scope and is recorded as an option for later work, with the three things it
would require: an enumerated action taxonomy for the target domain, a model trained for it, and
labelled screen recordings to train and evaluate against. None of the three exists in this tree, as
[current-state-capability-map.md §5](./current-state-capability-map.md) establishes for the model side
and [functional-spec.md §4](./functional-spec.md) records as the route's own prerequisites. Recording
that state is the honest treatment; describing how the gap might be closed would be inference the
dossier is written against.

## 6.4 Nothing is selected or obtained in this run

Naming the concrete candidate dependencies was the gap assessment's work and is already done in
[platform-capture-gap-assessment.md §6](./platform-capture-gap-assessment.md). This phase integrates
whatever a builder chooses from them, and the choice belongs to the builder because it turns on
requirements the request does not state — the recognition scope the content demands, the licences the
deployment can accept, the packaging weight it can carry, and the validation cost it can absorb.
Consistent with the read-only character of this entire dossier, no asset is selected, downloaded,
committed or benchmarked here.
