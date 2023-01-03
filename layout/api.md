## Public API
Use `internal` packages to reduce your public API surface


If your project contains multiple packages you may find you have some exported functions which are intended to be used by other packages in your project, but are not intended to be part of your project’s public API.
`internal/` give you possibility to place code which is public to your project, but private to other projects.

For example, a package `.../a/b/c/internal/d/e/f` can be imported only by code in the directory tree rooted at `/a/b/c`.
It cannot be imported by code in `/a/b/g` or in any other repository. 