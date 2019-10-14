# Plone

I'm not _that_ experienced in coding for Plone, so this is basically my collection of notes for things I need regularly or didn't know how to do.

## Access ZMI

Add `/manage` to the base URL.

## Allowing a content type to be displayed in a certain view

ZMI → `portal_types`, then select the type you want to change (e.g. _Folder_).
Then, add the name of your view (the `name` attribute you've used in `configure.zcml`) to the list of _Available view methods_.

This can also be done programmatically, but I don't know how (yet).
