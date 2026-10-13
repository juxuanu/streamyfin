# Bottom Tabs Report the Holder's Size but Dedupe on the Outer View's

**Date**: 2026-10-13
**Category**: ui
**Key files**: `patches/react-native-bottom-tabs@1.4.0.patch`,
`app/(auth)/(tabs)/_layout.tsx`

## Detail

On Android, `react-native-bottom-tabs` sizes every tab scene from an
`onNativeLayout` event carrying the inner `layoutHolder` dimensions, but its
layout-change listener only re-emits when the *outer* view's size changed.
Anything that shrinks the holder without touching the outer frame is reported
once and never corrected.

The keyboard does exactly that. Expo's manifest sets `adjustResize`, the app
runs edge-to-edge, and under that combination the deprecated system-window
inset still includes the IME, which Material's `BottomNavigationView` adds to
its bottom padding. The tab bar grows by the keyboard height, the holder
shrinks by the same amount, the outer view stays put. Opening the app from a
launcher search (keyboard already up) made the first report the shrunk height;
the keyboard then closed and the holder grew back silently, leaving the lower
part of every tab black until a rotation changed the outer size
(streamyfin/streamyfin#526).

The patch skips a layout taken while the keyboard is visible (so scenes keep
their size and the keyboard overlays them, as before) and keys the dedupe on
`layoutHolder`'s own size for the rest, so the layout after the keyboard
closes is reported. Upstream's unmerged fix
callstack/react-native-bottom-tabs#556 (issue #555) only re-keys the guard,
which would also make scenes shrink whenever the keyboard opens inside a tab;
that is why the patch is not a copy of it. Review the hunk when the library is
bumped.

## Symptom pattern

A fixed black or empty band at the bottom of every tab from launch, the exact
height of the keyboard, that rotating the phone fixes and a tab switch does
not. Any report of a tab screen "stuck at a smaller height" on Android is
this guard.
