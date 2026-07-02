# Embedding photos, exhibits, and collections

Kronofoto lets you drop live, always-current photo content into your own
website: a single photo, a search result / gallery, a saved list
("MyList"), or an Exhibit ("photostory"). It's **not** an `<iframe>` — it's
a small custom HTML element, `<fortepan-viewer>`, that renders directly
into your page (inside its own
[shadow DOM](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_shadow_DOM)
for style isolation, so it won't clash with your site's CSS).

## The short version

Every embeddable view has a "Share" / "Embed" option that generates a
two-line snippet:

```html
<script src="https://your-archive.example/static/kronofoto.js"></script>
<fortepan-viewer src="https://your-archive.example/..."></fortepan-viewer>
```

Paste both lines into your page's HTML wherever you want the widget to
appear. The `<script>` tag only needs to be included once per page, even if
you embed several `<fortepan-viewer>` elements.

!!! note "Prerequisite: set your site's domain"
    The `src` URL is built from Django's Sites framework
    (`Site.objects.get_current().domain`). If that's left at the default
    `example.com`, generated embed snippets will be broken. Set it under
    `/admin/sites/site/` before sharing any embed code.

## Embedding an Exhibit ("photostory")

1. Open the exhibit you want to embed.
2. Go to its embed page: `/exhibits/<id>/embed`.
3. Copy the generated snippet, e.g.:

   ```html
   <script src="https://your-archive.example/static/kronofoto.js"></script>
   <fortepan-viewer src="https://your-archive.example/exhibits/12-a-walk-through-1968"></fortepan-viewer>
   ```

See [`views.exhibit.embed`](../reference/views.md) and the
`exhibit-embed.html` template.

## Embedding a Collection ("MyList")

1. Open the collection (list) you own.
2. Go to its embed page: `/collections/<id>/embed`.
3. Copy the generated snippet. Collection embeds add a `constraint`
   attribute that scopes the viewer to just that collection's photos:

   ```html
   <script src="https://your-archive.example/static/kronofoto.js"></script>
   <fortepan-viewer
     src="https://your-archive.example/photos/random"
     constraint="collection:3f29c9c2-1234-4a5b-9abc-0123456789ab">
   </fortepan-viewer>
   ```

See [`views.collection.embed`](../reference/views.md) and the
`collection-embed.html` template.

## Embedding a search or a single photo

From any photo or search-results page, use the **Share** button — it opens
a popup with a "Copy" button that produces the same kind of snippet, using
the current photo/search as the `src` and, for a filtered search, the
search expression as `constraint` (e.g. `constraint="county:Jones"`).

See [`views.webcomponent.WebComponentPopupView`](../reference/views.md).

## The `<fortepan-viewer>` element

| Attribute    | Required | Meaning                                                                 |
| ------------ | -------- | ------------------------------------------------------------------------ |
| `src`        | yes      | The Kronofoto page to render inside the widget (a photo, exhibit, or `random-image` route). |
| `constraint` | no       | A [search expression](../reference/search.md) that scopes/filters what's shown — used for collections and filtered searches. |

Internally, `<fortepan-viewer>` fetches `src` with `Embedded: 1` (and
`Constraint: <value>` when set) request headers; the server responds with a
trimmed-down HTML fragment (no site header/footer) which gets loaded into
the element's shadow root, then HTMX/Alpine are re-initialized scoped to
that shadow root so the embedded widget stays interactive (pagination,
timeline scrubbing, etc.) without a full page reload.
