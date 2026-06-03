# Build output

AI agents write the generated Alleo E-widget here.

- `‹widget-name›.txt` — the widget itself: a complete HTML document saved with a
  `.txt` extension (the Alleo E-widget's file picker expects `.txt`).
- `‹widget-name›.README.md` — a build note describing the widget, the assumptions
  the agent made, its actions, and the widget settings you must enable.

## Using the output in Alleo

1. Open the **E-widget**'s **Settings → Source**.
2. Either choose **Custom HTML** and paste the contents of the `.txt` file, or
   choose **File** and upload the `.txt` file.
3. Enable the board features listed in the build note (actions, synced status,
   etc.).

This folder's generated files are ignored by git (except this README).
