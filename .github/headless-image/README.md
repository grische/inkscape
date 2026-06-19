# Headless Inkscape image (fork CI branch)

This branch (`ci/headless-inkscape-image`) is **not** an Inkscape source change.
It only carries the tooling to build a patched, headless Inkscape **container**:

- `Dockerfile` — builds Inkscape 1.4.2 (GTK3) from the upstream release tag with
  `0001-headless-recentmanager-5419.patch` applied.
- `0001-headless-recentmanager-5419.patch` — guards the headless-reachable
  `Gtk::RecentManager::get_default()` calls with `gdk_display_get_default()`, so
  Inkscape stops printing this on every command-line export:

  ```
  glibmm-WARNING **: Failed to wrap object of type 'GtkRecentManager'.
  ```

  Upstream issue: <https://gitlab.com/inkscape/inkscape/-/work_items/5419>.
  (The GTK4/`master` equivalent lives on `fix-5419-headless-recentmanager-warning`
  and uses `gtk_is_initialized()`, which does not exist in GTK3.)

The workflow `.github/workflows/build-headless-inkscape.yml` builds the image,
asserts the warning is gone, and pushes `ghcr.io/grische/inkscape:1.4.2-headless`.
It is consumed by <https://github.com/freifunk/device-pictures>.

## One-time setup on the fork

1. Enable **Actions** on the fork (Settings → Actions → General).
2. Run **Actions → Build headless Inkscape image → Run workflow** (~30–60 min).
3. Set the published GHCR package `inkscape` to **Public** (Packages → inkscape →
   Package settings → Change visibility) so `device-pictures` can pull it without
   credentials.
