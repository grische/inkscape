# Headless Inkscape image (fork CI branch)

This branch (`ci/headless-inkscape-image`) is **not** an Inkscape source change.
It only carries the tooling to build a patched, headless Inkscape **container**:

- `Dockerfile` — builds Inkscape 1.4.2 (GTK3) from the upstream release tag with
  both patches below applied.
- `0001-headless-recentmanager-5419.patch` — guards the headless-reachable
  `Gtk::RecentManager::get_default()` calls with `gdk_display_get_default()`, so
  Inkscape stops printing this on every command-line export:

  ```
  glibmm-WARNING **: Failed to wrap object of type 'GtkRecentManager'.
  ```

  Upstream issue: <https://gitlab.com/inkscape/inkscape/-/work_items/5419>.
  (The GTK4/`master` equivalent lives on `fix-5419-headless-recentmanager-warning`
  and uses `gtk_is_initialized()`, which does not exist in GTK3.)
- `0002-headless-pangoft2fontmap.patch` — moves the `Glib::wrap()` of the FT2 font
  map in `FontFactory` out of the constructor and into `get_font_families()`, the
  only consumer (a GUI-only path). The constructor no longer wraps, so the headless
  export path never triggers the warning; lifetime of the raw `fontServer` is now
  managed with an explicit `g_object_unref` in the destructor. Removes:

  ```
  ... WARNING **: Failed to wrap object of type 'PangoFT2FontMap'.
  ```

Both warnings share one root cause: headless = no display = the gtkmm/pangomm
type wrappers are never registered, so `Glib::wrap()` cannot find them.

The workflow `.github/workflows/build-headless-inkscape.yml` builds the image,
asserts no "Failed to wrap object" warning remains, and pushes
`ghcr.io/grische/inkscape:1.4.2-headless`. It is consumed by
<https://github.com/freifunk/device-pictures>.

## One-time setup on the fork

1. Enable **Actions** on the fork (Settings → Actions → General).
2. Run **Actions → Build headless Inkscape image → Run workflow** (~30–60 min).
3. Set the published GHCR package `inkscape` to **Public** (Packages → inkscape →
   Package settings → Change visibility) so `device-pictures` can pull it without
   credentials.
