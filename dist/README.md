# Build output

AI agents write the generated Alleo E-widget here.

- `‹widget-name›.Alleo-eWidget.txt` — the importable widget: a complete HTML
  document followed by a `WIDGETSETTINGS:` footer, saved with a `.txt` extension
  (the Alleo E-widget's file picker expects `.txt`).
- `‹widget-name›.README.md` — a build note describing the widget, the assumptions
  the agent made, its actions, and the widget settings you must enable.

## Using the output in Alleo

1. Open the **E-widget**'s **Settings → Source**.
2. Either choose **Custom HTML** and paste the contents of the
   `.Alleo-eWidget.txt` file, or choose **File** and upload the
   `.Alleo-eWidget.txt` file. Importing the file also applies the footer settings
   automatically.
3. Confirm the board features listed in the build note are enabled (actions,
   synced status, etc.).

This folder's generated files are ignored by git (except this README).
