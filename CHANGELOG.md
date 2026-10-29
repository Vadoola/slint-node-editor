# Changelog

## 2.0.0 — 2026-10-29

Requires Slint 1.18.0 or newer and Rust 1.92 or newer. Thanks to Olivier de
Gaalon for the layout corners, rank alignment, link accents and dimming, node
hover, undraggable nodes and the marquee fixes.

### Breaking changes and migration

- `LinkData` gains the fields `accent` and `dimmed`. A Rust struct literal that
  names every field must add both (`accent: 0, dimmed: false` keep the 1.x
  look) or end in `..Default::default()`; `LinkData::new`'s signature and the
  link it builds are unchanged. From 2.0, a generated struct may gain a field
  whose default keeps its behaviour in a minor release, so construct them
  through their constructors or `..Default::default()`
  ([compatibility and releases](CONTRIBUTING.md#compatibility-and-releases)).
- `GraphLogic::find_node_by_id` takes any `&impl Model<Data = T>` instead of
  only a `&VecModel<T>`. Calls that name its generic parameters, such as
  `find_node_by_id::<NodeData, _>(…)`, no longer compile: drop the turbofish
  and let inference choose them.
- `sugiyama_layout` returns each node's top-left corner, as documented; 1.x
  returned rust-sugiyama's centres, so cards of unequal size overlapped. Remove
  any half-size correction applied to its output. The layout starts at the
  origin, and each component's first rank starts at 0 on the layer axis.

### Added

- Node groups: `BaseGroup` and the default `GroupSheet`, rendered below the
  links for every row of `NodeEditor.groups`. Membership is `group-id` on the
  node rows; a sheet drags its members, and collapsing hides them through a
  visibility projection that also excludes them from marquee, pin and link
  picking. Selection gains `group-selected` and a five-argument
  `wire_selection!` arm; `wire_groups!` installs the host answers for the live
  drag preview and collapse. Rust: the `groups` module (`GroupMember`,
  `GroupModel`, `GroupLogic`, `Dragged`, `Bounds`), `GeometryCache` hidden
  nodes and group rectangles, and `NodeEditorSetup::with_drag_commit` for one
  drag commit across both gestures. New `groups` example.
- `BaseNode.draggable` and `MovableNode::draggable`, both default `true`. Off,
  a user drag moves the node only with its group. A drag starting on it does
  what one on empty canvas does (a marquee when `box-selection-on-empty` is
  on, nothing otherwise): no `start-node-drag` or `end-node-drag` is
  reported, and the release is no click. A press still selects it and its
  hover is unchanged. A drag of other selected nodes leaves it in place, in
  the live preview and in `GraphLogic::commit_drag` and
  `GroupLogic::commit_drag`; a group that moves still carries it. Bind the
  property from the row field your `MovableNode::draggable` reads, as you
  bind `selected`: the preview reads one and the commit the other. The trait
  method has a default, so existing implementations compile unchanged. Host
  writes to the rows, such as a layout or an undo, are not governed.
- `NodeEditorComputations.viewport-resized(width, height)` so hosts can track
  editor dimensions independently of grid updates, preserving the existing
  `viewport-changed` signature. Batched changes emit one deferred
  notification; hosts explicitly seed their initial size (#6).
- `SugiyamaConfig::rank_alignment` aligns a rank's nodes on their leading edge
  (`Start`) or their centre (`Center`, the default).
- `LinkData.accent` indexes `NodeEditor.link-accent-colors` from 1, a palette
  the application binds, and colours a link that has no known status; 0, the
  default, is no accent, and an index outside the palette leaves the link its
  `color`. The rule is `LinkColor.of`, exported for hosts that draw their own
  links.
- `LinkData.dimmed` draws a link faded to `NodeEditor.dimmed-link-alpha`, over
  whatever colour its status, accent or `color` gives it. Defaults to `false`.

### Changed

- A subclass's elements sit inside `BaseNode`'s drag area, so `hovered` and
  `NodeEditor.hovered-node-id` stay true while the pointer is over the node's
  own fields, labels and pins, and while a child holds a press (a link dragged
  from a pin, say) until its release. Content drawn outside the node's
  rectangle does not hover it. A press no child takes still selects or drags
  the node.
- `link-hovered()` also fires when the pointer leaves the canvas for a node
  drawn over it or for outside the window, with `hovered-link-id` back at -1;
  a link no longer stays hovered after the pointer left it that way.

### Fixed

- A marquee started on a node or group sheet (with the box-selection modifier,
  or on an undraggable node) was drawn and committed offset by the editor's
  position in its window.
- Share cancellation and gesture globals across the `@nodeeditor` module
  boundary so public removal and reset hooks cancel consumer interactions (#7).
- Cancel active forwarded marquees when retiring a pin or node, preventing a
  stale pointer release from committing the selection.

## 1.0.1 — 2026-09-17

Documentation-only patch. The library API and implementation are unchanged.

- Replace the broken README demo attachment with the verified GitHub release
  video and use explicit, versioned screenshot URLs on crates.io and docs.rs.
- Remove pre-publication setup instructions from the published README and guide.

## 1.0.0 — 2026-09-16

First stable release. Requires Slint 1.18.0 or newer and Rust 1.92 or newer.

- Distribute Slint components through the `@nodeeditor` library module.
  Consumers enable `experimental-module-builds` on slint-build and require
  Rust 1.92. Slint 1.18.0 is the minimum supported registry release.
- Routes update after programmatic geometry changes and drag commits; removal
  retires node and pin geometry. Picking uses the rendered world-space curve.
- Normalize connection direction before topology validation and choose the
  nearest eligible pin deterministically.
- Make layout input ordering and disconnected-component packing deterministic.
- Test public consumer gestures and configured marquee behavior.
- Focus the editor on canvas, node, pin, minimap, and reserved marquee presses,
  while preserving embedded text input and application-directed focus changes.
  Adapted from Olivier de Gaalon's focus fix.
- Reconcile rejected and snapped drag commits to the synchronous host model.
- Reject invalid controller viewport updates and require both normalization
  endpoints to exist. Add LinkData::new for explicitly colored links and
  standard Error support for ValidationError.

### Migration from earlier git snapshots

Windows using `wire_node_editor!` must expose
`public function focus-editor() { editor.focus(); }`. Manual integrations must
connect `NodeEditorInternalCallbacks.take-editor-focus` synchronously to that
function. See the integration guide for the complete window interface.

`Link.path-commands` is replaced by `Link.geometry: LinkPath`, containing commands
relative to its bounding box. Custom routes must supply that box.
`compute_link_path_callback` now takes a LinkPath constructor.

Use NodeEditorComputations for computations and NodeEditorEvents for BaseNode
double-clicks. Use the current quick-start macros and generated global exports;
removed component callbacks are not compatibility aliases. Selection remains
host-owned and must be projected synchronously before a drag proceeds.

### Release limitations

Independent editors in one window, complete keyboard accessibility, and
measured large-graph performance remain future work. See CONTRIBUTING.md for configuration coverage.
