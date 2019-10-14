# Plone

I'm not _that_ experienced in coding for Plone, so this is basically my collection of notes for things I need regularly or didn't know how to do.

## Access ZMI

Add `/manage` to the base URL.

## Allowing a content type to be displayed in a certain view

ZMI → `portal_types`, then select the type you want to change (e.g. _Folder_).
Then, add the name of your view (the `name` attribute you've used in `configure.zcml`) to the list of _Available view methods_.

This can also be done programmatically, but I don't know how (yet).

## Providing macros for several templates in a common file

Register a `.pt` file like so:

```xml
<browser:page
    for="*"
    name="ps.plone.mls.macros"
    template="templates/macros_p5.pt"
    permission="zope2.View"
    />
```

Then, use it by its name (here `ps.plone.mls.macros`) in other templates like so:

```xml
<div metal:use-macro="context/@@ps.plone.mls.macros/listing_results">Show list of listings.</div>
```

Thanks [Thomas](https://github.com/it-spirit) for showing me [how `ps.plone.mls`](https://github.com/propertyshelf/ps.plone.mls/blob/b902a800f8b0199d8a3d145f582471c38eaae015/src/ps/plone/mls/browser/configure.zcml#L73-L79) [does it](https://github.com/propertyshelf/ps.plone.mls/blob/b902a800f8b0199d8a3d145f582471c38eaae015/src/ps/plone/mls/browser/listings/templates/listing_results_p5.pt#L19)!
