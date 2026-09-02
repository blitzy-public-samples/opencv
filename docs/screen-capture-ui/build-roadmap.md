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
[§2](./platform-capture-gap-assessment.md) or [§3](./platform-capture-gap-assessment.md), which is
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

The consequence that shapes the design is that **the consent step shapes it even on the platforms
that do not need one.** The parity finding of
[platform-capture-gap-assessment.md §7](./platform-capture-gap-assessment.md) is that the three
targets differ not in whether pixels can be obtained but in who authorises the obtaining and how the
frames are then transported, and that one of the three mediates authorisation through a separate
service with no counterpart on the other two. A contract designed without that step, and later
retrofitted, either exposes the asymmetry to every caller on every platform or hides it and misleads
the caller on the platform that has it. So the contract carries an authorisation stage from the
start, and on a platform where authorisation is unconditional that stage succeeds immediately.

"Where feasible" is a real qualifier and not a hedge for its own sake. §5 records one bridge whose
verification the dossier could not complete, and the principle it works under is that a divergence
is recorded with its reason rather than asserted away.

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
surface can reach the capture API under conditions this dossier verified, and be displayed. Every
later phase consumes its output, so it is deliberately the narrowest phase in the roadmap: it proves
the acquisition-to-ingestion path and nothing else.

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

**Added dependencies:** the selected platform's acquisition mechanism, which is host code outside
this library and carries whatever the assessed mechanism itself requires; and, on the pipeline-string
route, the media-framework plugin that provides the screen-source element, which is a genuine added
package rather than a build flag. On the environment-mediated route the addition is a build whose
media backend includes device-opening support [modules/videoio/src/cap_ffmpeg_impl.hpp:1213] together
with the demuxer that acquisition mechanism is exposed as. Nothing is added to this library, and no
part of its source is changed.

**Sequencing:** none; this phase is first. It has no predecessor, and nothing in it waits on another
phase.

**Exit criteria:**

- The selected platform is recorded together with the evidence for its selection: which mechanism,
  which ingestion route, and the observation that both were available in the build under test.
- Frames from an explicitly named source arrive and are displayed. Named means the application states
  which surface it opened rather than accepting whatever an auto-detection sentinel resolves to.
- A request for an unavailable source fails explicitly, naming the source and the reason, and no
  other source is opened in its place.
- The route's build conditions are recorded as observed rather than as declared defaults, including
  the result of the backend-availability probe the public registry exposes
  [modules/videoio/include/opencv2/videoio/registry.hpp:45].
- The open and read timeouts are set explicitly through the parameter vector the open overloads accept
  [modules/videoio/include/opencv2/videoio.hpp:877,901], and the values used are recorded.
- Acceptance thresholds for capture frame rate and resolution are taken from user-supplied values
  where any exist. None were supplied with this request, so this exit is **recorded as blocked pending
  that product decision** rather than written against a figure this dossier would have had to invent.

## 2.1 The target platform is selected, not assumed

The phase's first activity is to select the platform, and the selection is an activity with an
evidence requirement rather than a decision this document makes. Nothing in the dossier ranks the
three targets, and ranking them here would substitute a preference for a finding.

The selection is gated on two conditions holding together for the same target. The platform must have
an acquisition mechanism the application can use in its intended deployment posture — attended or
unattended, whole-monitor or single-window, with or without an authorisation step — which is what
[platform-capture-gap-assessment.md §1](./platform-capture-gap-assessment.md),
[platform-capture-gap-assessment.md §2](./platform-capture-gap-assessment.md) and
[platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md) assess per platform.
And an ingestion route must carry that mechanism's output into the library, with the route's own
condition satisfied in the build under test, which is what
[technical-inventory.md §5](./technical-inventory.md) supplies the flags for and what
[platform-capture-gap-assessment.md §4](./platform-capture-gap-assessment.md) records as
route-by-route bridging.

Where a target satisfies the first condition but the bridge to an ingestion route was not established
— the case the gap assessment records for one platform and for one Windows mechanism — that target is
not eligible for this phase, because a prototype cannot rest on an unverified bridge. Recording that
ineligibility with its reason is part of this exit, and it is also the input Phase 4 (§5) picks up.

## 2.2 Route availability is a property of the build under test

The two routes are not interchangeable, and this phase records which one it used and why. The dossier
treats the pipeline-string route as the stronger of the two, because it is a documented parameter
contract on the open call [modules/videoio/include/opencv2/videoio.hpp:799-805] rather than an
environment variable, and because its condition is a naming requirement on the pipeline the
application writes [modules/videoio/src/cap_gstreamer.cpp:1343] rather than a build-time capability.
The other route reaches only what its input-format lookup resolves
[modules/videoio/src/cap_ffmpeg_impl.hpp:1206-1210] and needs device-opening support compiled in
[modules/videoio/src/cap_ffmpeg_impl.hpp:1213].

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
the common field set, the screenshot addressing and the durable append the contract is recorded
through. The frame-admission gate is implemented here too, because the stream records admitted frames
and the gate is what decides admission. Outside this phase's scope: any interface beyond the
preview Phase 1 established, and any extraction of content from the frames.

**Inputs:** the session clock and sequence design and the merge rule, from
[functional-spec.md §2](./functional-spec.md); the record taxonomy and the durability mechanism, from
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

**Sequencing:** after Phase 1, which supplies the frame stream. The gate compares a frame against the
previous retained frame, so it has nothing to compare until frames arrive.

**Exit criteria:**

- Every record kind carries a monotonic time value from the session's single clock and a sequence
  number unique within the session, and both are stamped at the moment of acquisition rather than at
  the moment of writing.
- The merge order is demonstrated on records of both kinds, including the equal-time rule: at equal
  monotonic time an input record sorts before a frame record, so the event that caused a change
  precedes the frame showing it, and equal time with equal kind falls through to the sequence number.
- An interrupted session leaves a readable stream, to exactly the guarantee
  [functional-spec.md §5](./functional-spec.md) states and under the assumptions it names (§3.3).
- The change gate is implemented to the score contract of
  [functional-spec.md §3](./functional-spec.md), with the per-pixel sensitivity, the admission
  threshold and the quiet-frame count that closes a segment configurable, and with the score's
  definition, range, whole-frame domain, first-frame rule and inclusive comparison fixed.
- Segment boundaries appear in the stream as records rather than being left for a consumer to
  re-derive.
- Where the backend supplies a presentation timestamp it is recorded on frame records as
  supplementary media metadata, and it is used for no ordering decision (§3.1).

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
[modules/videoio/src/plugin_capture_api.hpp:92,103]. The one readiness API requires every capture in
the call to share a backend and dispatches to a single backend, raising an error outside it
[modules/videoio/src/cap.cpp:629-652].

So there is no readiness signal on which to hang a change-driven capture loop, and the consequence is
architectural rather than inconvenient: the session polls, and the gate that decides which frames are
retained sits above the capture object in application code. The three operators a frame-to-frame gate
is composed from are not provided by the in-scope modules, as
[current-state-capability-map.md §2](./current-state-capability-map.md) records on the capability
side and [functional-spec.md §3](./functional-spec.md) on the requirement side; what the in-scope
modules do supply is the preprocessing and the interpretation of a difference image, inventoried in
[technical-inventory.md §2](./technical-inventory.md). The gate is therefore built in this phase, to
the score contract, and it is not configured into existence.

## 3.3 What "readable after interruption" means as an exit

The exit is stated against a mechanism because the format alone does not bound loss. A line-delimited
stream guarantees only that a complete line parses independently; a half-written record is invalid
regardless of how its fields are ordered.
[functional-spec.md §5](./functional-spec.md) therefore states the mechanism the guarantee rests on —
each record serialised in full, newline-terminated, written with a single append that is flushed
before the write counts as complete — and the guarantee that follows: every flushed record survives an
abrupt termination, at most the final unflushed record is lost, and no record depends on a later
record or on a session footer.

This phase's exit is met by demonstrating that guarantee on a session terminated without warning, and
by recording which of the two guarantees the host actually provides. Where append-and-flush durability
is unavailable the guarantee narrows to what the format supports on its own, and the phase records the
narrowed guarantee rather than claiming the stronger one.

# 4. UI Shell

Phase 3 of five. Capture and correlation now run; this phase gives a user something to watch, mark up
and control. It is the phase whose exits are most conditional, because a public declaration in the
display module does not imply that the active backend implements it.

**Scope:** preview, controls and annotation rendering. Preview is the continuous presentation of
captured frames; controls are session start and stop and the gate's sensitivity and threshold;
annotation rendering is the composition of marks, boxes and text onto the frame before it is
displayed. Outside this phase's scope: the persistent annotation model, which
[functional-spec.md §6](./functional-spec.md) assigns to the application and
[functional-spec.md §5](./functional-spec.md) specifies as revision records in the same stream, and
any second platform.

**Inputs:** the display and interaction verdicts with the backend condition each carries, assessed in
[current-state-capability-map.md §3](./current-state-capability-map.md) and inventoried in
[technical-inventory.md §3](./technical-inventory.md); the interface requirements and their verdicts,
from [functional-spec.md §6](./functional-spec.md); and the finding that display and interaction
parity is backend-conditional rather than platform-conditional, from
[platform-capture-gap-assessment.md §7](./platform-capture-gap-assessment.md).

**Added dependencies:** none beyond the display backend already present in the build Phase 1 used —
unless a control this phase requires exists on one backend only, which would add one. The button
control is declared for a single toolkit [modules/highgui/include/opencv2/highgui.hpp:808-810] whose
gate carries a declared default of OFF [CMakeLists.txt:299], so a control surface that depends on it
adds that toolkit to every deployment. A control surface built from trackbars and keyboard input adds
nothing, and is preferred for that reason under §1.2.

**Sequencing:** after Phase 2, which supplies both the frames the shell previews and the timeline it
presents.

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

## 4.1 Every display and interaction exit is backend-conditional

Runtime backend membership is a fixed list built into the module
[modules/highgui/src/registry.impl.hpp:27-66]: the GTK family, a framebuffer backend, a Windows
backend compiled only on that platform, and one further entry that is present in the source but
disabled behind a compile-time guard [modules/highgui/src/registry.impl.hpp:51] — what the guard
shows is work started or considered and left incomplete, and the code records no reason. Which member
is active in a given build is not decidable from this document, which is why the probe result is an
exit criterion rather than a premise.

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

Which toolkit is not decided here, and this document names none: no interface toolkit or component
library is specified for this application, and naming one would be an invention rather than a
requirement — the same failure as asserting a capability no source establishes. The phase exit is met
by a shell that works on the backend it names, with the features the backend does not support
reported as unavailable; choosing a toolkit for a production interface is a decision outside this
roadmap.

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
additions are the portal client that negotiates the session and the media-transport client that
carries the frames, both host-side; the display backend for that target is a separate matter and is
absent from runtime
membership in this tree [modules/highgui/src/registry.impl.hpp:27-66], with its build gate carrying a
declared default of OFF [CMakeLists.txt:235] (§5.3).

**Sequencing:** after Phase 3, so that the contract being held constant already exists and has been
exercised end to end on one platform. Attempting parity before there is something to be at parity
with produces three implementations and no contract.

**Exit criteria:**

- Each additional platform reaches the same application-side contract, **or** its divergence is
  recorded with the reason and with what the divergence costs a caller.
- The authorisation path is exercised on the consent-mediated platform, including a restored session
  rather than only a first run, since the two differ in whether a user interaction occurs.
- Where the bridge from that platform's mediated transport into an ingestion route cannot be
  verified, the phase **records it as unresolved** and does not assert parity for that platform
  (§5.2).
- The deployment posture each platform actually supports is recorded — attended or unattended, and
  whole-monitor or window-scoped — rather than assumed uniform across the three.
- Display and interaction parity is reported separately from capture parity, per backend rather than
  per platform (§5.3).

## 5.1 What parity is, and what it is not

The parity substance, so that this phase has a criterion rather than an aspiration: the platforms
differ **not in whether pixels can be obtained but in who authorises the obtaining and how the frames
are transported**, which is the comparison
[platform-capture-gap-assessment.md §7](./platform-capture-gap-assessment.md) draws across the three
assessments. The X11 path admits an unmediated read
[platform-capture-gap-assessment.md §2](./platform-capture-gap-assessment.md); the Windows path is
likewise unmediated but binds its modern mechanisms to a graphics device
[platform-capture-gap-assessment.md §1](./platform-capture-gap-assessment.md); and the Wayland path
mediates authorisation through a separate service and delivers frames over a separate transport
[platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md).

What follows for the contract is the asymmetry §1.1 already committed to: a portable abstraction must
accommodate an authorisation step that has no counterpart on the other two targets. That is why this
phase's exit is about the contract holding rather than about the mechanisms converging, and why the
posture each platform supports is recorded rather than presumed — an application designed around
unattended operation on one target may require a user interaction on another, and that is a
contract-visible difference even when the frames are identical.

The mechanisms themselves, and the sources establishing them, are not restated here. They belong to
the three platform sections, which is the single-ownership rule this document works under, and a
second account of a mechanism is exactly where its condition gets dropped.

## 5.2 The consent-mediated bridge is an open dependency

This phase carries one unresolved dependency, and writing it as resolved would be the dossier's worst
failure mode.
[platform-capture-gap-assessment.md §3](./platform-capture-gap-assessment.md) records that the bridge
from a mediated screencast session into an ingestion route has no sourced, working contract: carrying
the session's transport handle and the stream's target identity into a media-framework element
implies an owner for the session's lifetime, a transfer mechanism for the handle, negotiation on the
library side, and an added source dependency, and the dossier established none of that chain.

So this phase does not treat parity on that platform as achievable by construction. Its work there
begins with establishing the bridge, and its exit admits three outcomes: the bridge is established
and the contract holds; the bridge is established and the contract holds with a recorded divergence;
or the bridge is not established and the phase records that as unresolved, leaving the platform
outside the parity claim. The third outcome is a legitimate result of this phase, not a failure of
it, and Phase 1's platform selection (§2.1) is what keeps the roadmap from having depended on it
earlier.

## 5.3 Display parity is a separate question from capture parity

The two are independent and are reported separately. Capture parity is about acquisition mechanisms
and ingestion routes; display parity is about which runtime display backend is available on each
platform and which operations that backend implements — the condition §4 established, which is a
property of the backend rather than of the platform.

The two questions can diverge on the same target: a platform may have a verified capture path and no
display backend of the same name in this tree's runtime membership
[modules/highgui/src/registry.impl.hpp:27-66], in which case the session captures there and previews
through a different backend, with the interaction features that backend does not implement reported
unavailable. Reporting one number for "platform support" would hide exactly that case, which is why
the exit asks for a per-backend report.

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

**Added dependencies:** on the in-library route, the model assets and the vocabulary — artefacts
rather than a library, which is why §1.2 prefers this route. On the alternative route, an external
recognition engine and its language data, which adds a native library with its own licensing,
packaging and build integration. Action extraction on the two routes this phase implements adds
nothing: it consumes the event stream Phase 2 already produces.

**Sequencing:** last. Text extraction needs admitted frames and their changed regions; action
extraction needs the correlated timeline; and both write into a record taxonomy Phase 2 established.

**Exit criteria:**

- **Text exit.** The chosen assets are integrated, the vocabulary is supplied to the recogniser, and
  the result is evaluated against a labelled sample of the actual screen content the application
  targets rather than a general document corpus. The accuracy threshold is taken from a user-supplied
  value where one exists; none was supplied with this request, so this exit is **recorded as blocked
  pending that product decision**.
- **Action exit.** The aggregation taxonomy is enumerated and closed, each entry's rule is defined,
  confidence is recorded as a rule-satisfaction indicator rather than a probability, and the outputs
  are written into the note contract. Action-coverage thresholds are the same kind of product
  decision, so where none is supplied that part of the exit is **recorded as blocked pending that
  decision** while the taxonomy and rule definitions remain evaluable as written.
- Every asset carries a recorded provenance and licence, since model licensing varies independently
  of this library's.
- Extraction failure is demonstrated to be independent of capture: a frame whose extraction failed is
  still an admitted frame and still produces a record, with the four-state extraction status
  [functional-spec.md §5](./functional-spec.md) specifies.

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
