
# Pinch->(Zoom|Scale)
[Desktop] New flag to divert touchpad-pinch from viewport-zoom to text-scale

### FLOW CHART

```
INPUT (touchpad-pinch event)
  ↓         ,-new-----------------------,
COMPOSITOR  | <=>  FLAG  <=>  FLAGS-UI  |
  ↓         '---------------------------'
  ↓
  +-> DEFAULT OUTPUT (viewport-zoom)
  ↓
,-new-----------------------------------,
| BROWSER -> ALT OUTPUT (text-scale)    |
'---------------------------------------'
```

Viewport-zoom is done in directly in compositor, while text-scale needs the
browser. This flow avoids unnecessary IPC.



# Practical steps

### First patch
Add name+email to `src/+/HEAD/AUTHORS` (inside the same commit)


### RENDERER EDITS

<details><summary><code>cc/input/input_handler.h</code>
</summary>Declare new virtual method in the delegate/interface<br><br></details>

<details><summary><code>cc/input/input_handler.cc</code>
</summary>Implement new delegate method (call it from PinchGesture*)<br><br></details>

### BROWSER EDITS

<details><summary><code>content/browser/renderer_host/render_widget_host_impl.h</code>
</summary>Declare new method RequestPageZoomByDelta<br><br></details>

<details><summary><code>content/browser/renderer_host/render_widget_host_impl.cc</code>
</summary>Implement new method RequestPageZoomByDelta<br><br></details>

<details><summary><code>content/browser/renderer_host/render_widget_host_view_base.cc</code>
</summary>Define new method in delegate<br><br></details>

### FLAGS EDITS

<details><summary><code>content/public/common/content_features.cc</code>
</summary>Add a <code>BASE_FEATURE();</code><br><br></details>

<details><summary><code>content/public/common/content_features.h</code>
</summary>Add a <code>CONTENT_EXPORT BASE_DECLARE_FEATURE();</code><br><br></details>

<details><summary><code>chrome/browser/about_flags.cc</code>
</summary>Add a <code>{"feature-name", flag_descriptions::}</code> (before "Add new entries above this line")<br><br></details>

<details><summary><code>chrome/browser/flag_descriptions.h</code>
</summary>Add an <code>inline constexpr char kFeatureName[]</code><br><br></details>

<details><summary><code>chrome/browser/flag-metadata.json</code>
</summary>Add <code>{"name", "owners", "expiry_milestone"}</code><br><br></details>

<details><summary><code>tools/metrics/histograms/enums.xml</code>
</summary>Run cmds (see build notes)<br><br></details>


<br/>

<details><summary>Build notes (cmds)</summary>

- To check `flag-metadata.json`:
  - `./out/Default/unit_tests --gtest_filter=AboutFlagsTest.EveryFlagHasMetadata`
- To make `enums.xml`:
  - Add `./tools/metrics/histograms/generate_flag_enums.py --feature <your awesome feature>`
  - Check `./out/Default/unit_tests --gtest_filter=AboutFlagsHistogramTest.CheckHistograms`
  - Fix `git cl format` (+ recheck)

</details>
