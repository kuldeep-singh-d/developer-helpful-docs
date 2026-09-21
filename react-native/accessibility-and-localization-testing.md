# React Native Accessibility and Localization Testing

A practical guide for finding screen-reader, font-scaling, focus, touch-target, reduced-motion, right-to-left, translation, and locale-formatting problems before release.

## Test accessibility and localization together

Both areas expose assumptions that are easy to miss on a default English development device:

- Text may become longer or display right to left.
- Large fonts can hide controls or break fixed layouts.
- Screen readers use semantic order rather than visual appearance alone.
- Dates, numbers, names, addresses, and currencies vary by locale.
- Color, motion, gestures, and time limits may block some users.

Automated checks help, but important flows must be tested with platform accessibility services and real localized content.

## Define a representative test matrix

At minimum, cover:

| Dimension | Useful cases |
|---|---|
| Screen reader | Android TalkBack and iOS VoiceOver |
| Text size | Default and largest supported accessibility size |
| Display | Small screen, landscape where supported, zoomed display |
| Motion | Reduced-motion preference enabled |
| Contrast/color | Dark mode, increased contrast, color-blind-safe meaning |
| Direction | Left-to-right and right-to-left |
| Language | Default language plus longest/most important translations |
| Formatting | Different date, number, currency, and plural rules |

Prioritize languages and device settings using actual audience and product requirements.

## Start with semantic controls

Use built-in interactive components or expose a correct role, name, state, and action.

```tsx
<Pressable
  accessibilityRole="button"
  accessibilityLabel="Save profile"
  accessibilityHint="Saves your changes"
  onPress={saveProfile}>
  <Text>Save</Text>
</Pressable>
```

If visible text already provides a clear accessible name, avoid repeating unnecessary labels. Accessibility hints should explain the result only when the result is not obvious.

For custom controls, expose state:

```tsx
<Pressable
  accessibilityRole="switch"
  accessibilityLabel="Notifications"
  accessibilityState={{checked: notificationsEnabled}}
  onPress={toggleNotifications}>
  <Text>{notificationsEnabled ? 'On' : 'Off'}</Text>
</Pressable>
```

Do not make a visual `View` behave like a button without keyboard/screen-reader semantics, state, disabled behavior, and a sufficiently large touch target.

## Test with TalkBack and VoiceOver

For every critical screen:

1. Navigate from the top using swipe gestures only.
2. Confirm focus order matches the intended reading/task order.
3. Verify each control has a concise name, correct role, state, and action.
4. Confirm decorative images are ignored and meaningful images are described.
5. Activate controls without relying on visual location.
6. Complete forms and recover from validation errors.
7. Open and close dialogs, menus, sheets, and navigation transitions.

Test on a physical device when possible. Simulator/desktop screen-reader behavior may differ.

## Manage focus after interface changes

Focus should move deliberately when:

- A modal or dialog opens.
- A blocking error appears.
- A screen changes after navigation.
- A list item is added or removed.
- A loading state replaces content.

Avoid moving focus for every minor update. Unexpected focus changes disorient users and may repeatedly announce content.

When a modal closes, return focus to the control that opened it when practical.

## Make validation errors discoverable

- Keep the entered value when validation fails.
- Explain the problem and how to correct it.
- Associate each error with its field.
- Announce important new errors to assistive technology.
- Move focus to an error summary or first invalid field only when it helps the flow.
- Do not rely on red color alone.

Use secure input settings for secrets, but ensure password requirements and show/hide controls remain understandable.

## Support large text without clipping

Avoid fixed-height containers around user-visible text. Test:

- Buttons with two-line labels.
- Navigation titles and tabs.
- Form labels, errors, and helper text.
- Cards containing translated content.
- Bottom sheets and dialogs.
- Empty, loading, and error states.

Do not disable font scaling globally just to preserve a layout. If a specific control needs a justified limit, document and test the decision.

```bash
# Read the current Android system font scale from a connected device/emulator.
adb shell settings get system font_scale

# Set a larger Android font scale for testing.
adb shell settings put system font_scale 1.5

# Restore the common default Android font scale after testing.
adb shell settings put system font_scale 1.0
```

> [!WARNING]
> These commands change a device-wide setting. Record the original value and restore that value after testing, especially on a personal device.

## Provide usable touch targets and gestures

- Make controls large enough for reliable touch interaction.
- Leave spacing between adjacent destructive and primary actions.
- Provide a simple alternative to complex swipes, drag-and-drop, or multi-finger gestures.
- Do not require precise timing or repeated rapid taps.
- Ensure an icon-only control has an accessible name.
- Test one-handed use and small screens.

Use `hitSlop` carefully when expanding a touchable area; overlapping touch targets can send an action to the wrong control.

## Do not rely on color, sound, or motion alone

Pair color with text, shape, iconography, or state. Examples:

- Show an error icon and message, not only a red border.
- Add text labels to chart categories where possible.
- Provide captions/transcripts for meaningful audio/video.
- Give visual feedback for sound-only alerts.
- Respect reduced-motion preferences for nonessential animations.

```ts
import {AccessibilityInfo} from 'react-native';

const reduceMotionEnabled = await AccessibilityInfo.isReduceMotionEnabled();

if (reduceMotionEnabled) {
  showResultWithoutDecorativeAnimation();
} else {
  playResultAnimation();
}
```

Keep critical state changes understandable even when animation is removed.

## Externalize user-visible text

Do not assemble translated sentences from fragments:

```ts
// Fragile: grammar and word order cannot adapt safely to every language.
const message = translatedCount + ' ' + translatedItems;
```

Use complete messages with placeholders and plural rules provided by the chosen localization system. Keep developer logs, analytics keys, and user-visible translations separate.

Include accessibility labels, error messages, notification text, permission explanations, widgets, and store-facing strings in the localization inventory.

## Format locale-sensitive values correctly

Use locale-aware formatters rather than manual concatenation:

```ts
const price = new Intl.NumberFormat(locale, {
  style: 'currency',
  currency: currencyCode,
}).format(amount);

const dateLabel = new Intl.DateTimeFormat(locale, {
  dateStyle: 'medium',
}).format(date);
```

Confirm the JavaScript runtime and supported platforms provide the required `Intl` behavior or include an evaluated fallback.

Test:

- Decimal and thousands separators.
- Currency symbol placement and currency precision.
- 12-hour and 24-hour time preferences.
- Calendar date versus instant/time-zone behavior.
- Singular, plural, zero, and other grammatical categories.
- Names and addresses without assuming one global format.

Never use a formatted local date string as a stable database or API identifier.

## Design for right-to-left layouts

Prefer logical layout concepts such as start/end where supported instead of hard-coded left/right. Review:

- Navigation arrows and directional icons.
- Row order and alignment.
- Mixed-direction content such as phone numbers, codes, and URLs.
- Charts, timelines, progress, and media controls.
- Gestures whose direction has product meaning.

```ts
import {I18nManager} from 'react-native';

const isRightToLeft = I18nManager.isRTL;
```

Do not mirror every icon automatically. A back arrow may mirror, while a play icon, brand mark, or real-world clockwise symbol often should not.

Some RTL configuration changes require an application restart. Verify cold launch as well as fast refresh.

## Handle missing and long translations

- Define a deliberate fallback locale.
- Detect missing keys in development and CI.
- Avoid silently shipping raw translation keys.
- Test strings expanded well beyond English length.
- Allow buttons, tabs, and cards to wrap or adapt.
- Keep placeholders and markup validated across locales.
- Give translators context, screenshots, and placeholder descriptions.

Pseudo-localization is useful for exposing hard-coded text, truncation, unsupported characters, and layout assumptions before translations are complete.

## Test dynamic and server-provided content

Localization is not limited to strings bundled with the app. Verify:

- Server errors are safe, understandable, and localized at the correct layer.
- User-generated text supports mixed scripts and does not break layouts.
- Remote configuration does not insert unreviewed English into localized screens.
- Push notifications and email/deep-link destinations use consistent locale rules.
- Content fallback is explicit when a translation is unavailable.

Do not render raw backend exception messages directly to users.

## Add accessibility-oriented automated tests

Component tests can verify names, roles, state, and user interaction:

```tsx
const saveButton = screen.getByRole('button', {name: 'Save profile'});

expect(saveButton).toBeEnabled();
await user.press(saveButton);
```

Automated rules can catch missing properties and some contrast/structure problems, but they cannot judge whether focus order, wording, gesture alternatives, or a full screen-reader journey is understandable.

## Include accessibility and locale cases in CI

Useful automated checks include:

- Missing or unused translation keys.
- Placeholder mismatch between locales.
- Component behavior queried by accessible role/name.
- Screenshots for representative long-text and RTL states.
- Critical E2E flow in at least one non-default locale.
- Lint rules for hard-coded user-visible text where appropriate.

Review screenshot changes instead of accepting all baseline updates automatically.

## Release checklist

- [ ] Critical journeys work with TalkBack and VoiceOver.
- [ ] Focus order and modal focus are predictable.
- [ ] Controls expose correct names, roles, states, and actions.
- [ ] Forms announce and explain errors.
- [ ] Largest supported text size does not hide essential actions.
- [ ] Meaning is not conveyed by color, sound, gesture, or motion alone.
- [ ] Reduced-motion behavior is tested.
- [ ] RTL layout and mixed-direction text are tested.
- [ ] Dates, numbers, currencies, and plurals use locale-aware formatting.
- [ ] Missing keys and long translations have been checked.
- [ ] Core flows are tested on real platform accessibility services.

## Related guides

- [Device and OS Fragmentation Testing](device-and-os-fragmentation-testing.md)
- [Mobile Testing and CI Strategy](mobile-testing-and-ci-strategy.md)
- [Mobile Permissions Troubleshooting](mobile-permissions-troubleshooting.md)
- [App Store Submission and Release Management](app-store-submission-and-release-management.md)

## Official references

- [React Native Accessibility](https://reactnative.dev/docs/accessibility)
- [React Native AccessibilityInfo](https://reactnative.dev/docs/accessibilityinfo)
- [React Native I18nManager](https://reactnative.dev/docs/i18nmanager)
- [Android Accessibility Testing](https://developer.android.com/guide/topics/ui/accessibility/testing)
- [Apple Accessibility](https://developer.apple.com/accessibility/)
