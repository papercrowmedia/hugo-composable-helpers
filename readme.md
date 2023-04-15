# Hugo Composable Helpers

A small set of partials to help compose your Hugo layouts.

## Installation

Requirements:

- Go 1.14
- Hugo 0.61.0

If you havent already, [init](https://gohugo.io/hugo-modules/use-modules/#initialize-a-new-module) your project as Hugo Module:

```shell
hugo mod init
```

Then you will need to add this module to your project's module imports:

```yaml
# config.yaml

module:
  imports:
    - path: github.com/papercrowmedia/hugo-composable-helpers
```

## Partials

### `-context`

Takes a 2 item `slice` and returns a `dict` with `.Content` and `.Params` keys.

```
{{ with partial "-context" (slice "Content" (dict "a" 1)) }}
  {{ .Content }} = "Content"
  {{ .Params }} = (dict "a" 1)
{{ end }}
```

```
{{ with partial "-context" (dict "a" 1) }}
  {{ .Content }} = ""
  {{ .Params }} = (dict "a" 1)
{{ end }}
```

```
{{ with partial "-context" "Content" }}
  {{ .Content }} = "Content"
  {{ .Params }} = (dict)
{{ end }}
```

### `-split`

Splits a `dict` into two based on the `slice` of keys. The result will be a two element slice where the first index is the original dict without the keys. The second index is a dict with the keys that were removed.

```go
{{ $params := dict
  "a" 1
  "b" 2
  "c" 3
}}

{{ with partial "-split" (slice $params (slice "a" "c"))}}
  {{ index . 0 }} = {{ dict "b" 2 }}
  {{ index . 1 }} = {{ dict "a" 1 "c" 3 }}
{{ end }}
```

### `-call`

Takes a `slice` of 2 elements where the first is the name of the partial to call with a `.Content` and `.Params` context. The second item is passed to `-context` to be used as context.

```go
{{ define "partials/with-content-and-params" }}
  {{ . }} = {{ dict "Content" "some-content" "Params" (dict "a" 1) }}
{{ end }}

{{ partial "-call" (slice "with-content-and-params" (slice "some content" (dict "a" 1)))}}
```

```go
{{ define "partials/with-content-and-params" }}
  {{ . }} = {{ dict "Content" "some-content" "Params" (dict) }}
{{ end }}

{{ partial "-call" (slice "with-content" "some content")}}
```

### `-element`

Dnymaically compose HTML tags based on data. The context is parsed with `-context`

```go
{{ partial "-element" (slice "Hello World" (dict "class" "bg-red-200"))}}
```

```html
<div class="bg-red-200">Hello World</div>
```

```go
{{ partial "-element" (slice "Hello World" (dict "tag" "span" "class" "bg-red-200"))}}
```

```html
<span class="bg-red-200">Hello World</span>
```

It even handles for self closing tags.

```go
{{ partial "-element" (slice "Hello World" (dict "tag" "img" "class" "bg-red-200" "src" "image.jpg"))}}
```

```html
<img class="bg-red-200" src="img.jpg" />
```

You can use the `-element` partial to compose inside out layouts.

```go
{{ partial "-element" (dict "tag" "img" "class" "w-full h-full object-cover" "src" "image.jpg")}}
  {{ partial "-element" (slice . (dict "class" "aspect-w-2 aspect-h-1"))}}
{{ end }}
```

```html
<div class="aspect-w-2 aspect-h-1">
  <img class="w-full h-full object-cover" src="img.jpg" />
</div>
```

### `-html-attrs`

Compose dynamic html attributes as a `dict` instead of complex and messy `if` logic.

```
{{ $class := "bg-red-200" }}

{{ with partial "-html-attrs" (dict "class" (slice "sm:px-8" $class)) }}
  <div {{ . }}></div>
{{ end }}
```
