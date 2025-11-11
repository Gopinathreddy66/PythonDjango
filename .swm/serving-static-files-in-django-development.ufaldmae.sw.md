---
title: Serving Static Files in Django Development
---
# introduction

This document explains how static files are served during Django development. We will cover:

1. Why static files are served only in development and not production.
2. How the serve function processes and validates file paths.
3. How directory indexes are optionally generated.
4. How HTTP caching headers are handled to optimize file serving.

# why serve static files only in development

The static file serving code is explicitly designed for development use only. It is not optimized or secure enough for production environments. This is stated clearly in the file <SwmPath>[django/views/static.py](django/views/static.py)</SwmPath>, which contains the views and functions for serving static files during development. Production setups should use dedicated web servers or CDN solutions for static content delivery.

# how the serve function processes requests

The main entry point is the serve function. It takes a request, a relative file path, a document root directory, and an optional flag to show directory indexes.

The function first normalizes and sanitizes the requested path to prevent directory traversal attacks by stripping empty components, drive letters, and special path parts like '.' and '..'. If the normalized path differs from the original, it redirects to the normalized path to maintain consistent URLs.

Next, it constructs the full filesystem path by joining the document root with the sanitized path. If this path is a directory and <SwmToken path="django/views/static.py" pos="16:15:15" line-data="def serve(request, path, document_root=None, show_indexes=False):">`show_indexes`</SwmToken> is True, it returns a directory listing. Otherwise, it raises a 404 error for directories or missing files.

<SwmSnippet path="/django/views/static.py" line="12">

---

This flow ensures only valid, safe file paths under the document root are served.

```python
from django.http import Http404, HttpResponse, HttpResponseRedirect, HttpResponseNotModified
from django.template import loader, Template, Context, TemplateDoesNotExist
from django.utils.http import http_date, parse_http_date

def serve(request, path, document_root=None, show_indexes=False):
    """
    Serve static files below a given point in the directory structure.

    To use, put a URL pattern such as::

        (r'^(?P<path>.*)$', 'django.views.static.serve', {'document_root' : '/path/to/my/files/'})

    in your URLconf. You must provide the ``document_root`` param. You may
    also set ``show_indexes`` to ``True`` if you'd like to serve a basic index
    of the directory.  This index view will use the template hardcoded below,
    but if you'd like to override it, you can create a template called
    ``static/directory_index.html``.
    """
    path = posixpath.normpath(urllib.unquote(path))
    path = path.lstrip('/')
    newpath = ''
    for part in path.split('/'):
        if not part:
            # Strip empty path components.
            continue
        drive, part = os.path.splitdrive(part)
        head, part = os.path.split(part)
        if part in (os.curdir, os.pardir):
            # Strip '.' and '..' in path.
            continue
        newpath = os.path.join(newpath, part).replace('\\', '/')
    if newpath and path != newpath:
        return HttpResponseRedirect(newpath)
    fullpath = os.path.join(document_root, newpath)
    if os.path.isdir(fullpath):
        if show_indexes:
            return directory_index(newpath, fullpath)
        raise Http404("Directory indexes are not allowed here.")
    if not os.path.exists(fullpath):
        raise Http404('"%s" does not exist' % fullpath)
    # Respect the If-Modified-Since header.
    statobj = os.stat(fullpath)
```

---

</SwmSnippet>

# how directory indexes are generated

If directory indexes are enabled and the requested path is a directory, the <SwmToken path="django/views/static.py" pos="28:4:4" line-data="    ``static/directory_index.html``.">`directory_index`</SwmToken> function is called.

This function tries to load a user-provided template named <SwmToken path="django/views/static.py" pos="28:2:6" line-data="    ``static/directory_index.html``.">`static/directory_index.html`</SwmToken> or <SwmToken path="django/views/static.py" pos="28:2:4" line-data="    ``static/directory_index.html``.">`static/directory_index`</SwmToken>. If none is found, it falls back to a hardcoded default HTML template embedded in the code.

It then lists all non-hidden files and directories in the target directory, appending a slash to directory names. This list and the directory path are passed to the template context to render a simple clickable index page.

<SwmSnippet path="/django/views/static.py" line="67">

---

This approach allows developers to customize directory listings or rely on a basic default.

```python
DEFAULT_DIRECTORY_INDEX_TEMPLATE = """
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
  <head>
    <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta http-equiv="Content-Language" content="en-us" />
    <meta name="robots" content="NONE,NOARCHIVE" />
    <title>Index of {{ directory }}</title>
  </head>
  <body>
    <h1>Index of {{ directory }}</h1>
    <ul>
      {% ifnotequal directory "/" %}
      <li><a href="../">../</a></li>
      {% endifnotequal %}
      {% for f in file_list %}
      <li><a href="{{ f|urlencode }}">{{ f }}</a></li>
      {% endfor %}
    </ul>
  </body>
</html>
"""

def directory_index(path, fullpath):
    try:
        t = loader.select_template(['static/directory_index.html',
                'static/directory_index'])
    except TemplateDoesNotExist:
        t = Template(DEFAULT_DIRECTORY_INDEX_TEMPLATE, name='Default directory index template')
    files = []
    for f in os.listdir(fullpath):
        if not f.startswith('.'):
            if os.path.isdir(os.path.join(fullpath, f)):
                f += '/'
            files.append(f)
    c = Context({
        'directory' : path + '/',
        'file_list' : files,
    })
    return HttpResponse(t.render(c))
```

---

</SwmSnippet>

# how HTTP caching headers are handled

To avoid unnecessary file transfers, the serve function respects the <SwmToken path="django/views/static.py" pos="52:7:11" line-data="    # Respect the If-Modified-Since header.">`If-Modified-Since`</SwmToken> HTTP header.

It obtains the file's modification time and size, then calls <SwmToken path="django/views/static.py" pos="56:5:5" line-data="    if not was_modified_since(request.META.get(&#39;HTTP_IF_MODIFIED_SINCE&#39;),">`was_modified_since`</SwmToken> to compare these against the header value.

The <SwmToken path="django/views/static.py" pos="56:5:5" line-data="    if not was_modified_since(request.META.get(&#39;HTTP_IF_MODIFIED_SINCE&#39;),">`was_modified_since`</SwmToken> function parses the header, checking both the timestamp and optional content length. If the file has not changed since the client last fetched it, the function returns False, prompting serve to return an <SwmToken path="django/views/static.py" pos="12:17:17" line-data="from django.http import Http404, HttpResponse, HttpResponseRedirect, HttpResponseNotModified">`HttpResponseNotModified`</SwmToken> with the correct MIME type.

If the file is modified or the header is <SwmPath>[django/…/data/invalid/](django/contrib/gis/tests/data/invalid/)</SwmPath>, serve reads the file content, guesses its MIME type and encoding, and returns it with appropriate <SwmToken path="django/views/static.py" pos="60:4:6" line-data="    response[&quot;Last-Modified&quot;] = http_date(statobj.st_mtime)">`Last-Modified`</SwmToken> and <SwmToken path="django/views/static.py" pos="61:4:6" line-data="    response[&quot;Content-Length&quot;] = statobj.st_size">`Content-Length`</SwmToken> headers.

<SwmSnippet path="/django/views/static.py" line="54">

---

This mechanism reduces bandwidth and speeds up page loads during development.

```python
    mimetype, encoding = mimetypes.guess_type(fullpath)
    mimetype = mimetype or 'application/octet-stream'
    if not was_modified_since(request.META.get('HTTP_IF_MODIFIED_SINCE'),
                              statobj.st_mtime, statobj.st_size):
        return HttpResponseNotModified(mimetype=mimetype)
    response = HttpResponse(open(fullpath, 'rb').read(), mimetype=mimetype)
    response["Last-Modified"] = http_date(statobj.st_mtime)
    response["Content-Length"] = statobj.st_size
    if encoding:
        response["Content-Encoding"] = encoding
    return response
```

---

</SwmSnippet>

<SwmSnippet path="/django/views/static.py" line="108">

---

&nbsp;

```python
def was_modified_since(header=None, mtime=0, size=0):
    """
    Was something modified since the user last downloaded it?

    header
      This is the value of the If-Modified-Since header.  If this is None,
      I'll just return True.

    mtime
      This is the modification time of the item we're talking about.

    size
      This is the size of the item we're talking about.
    """
    try:
        if header is None:
            raise ValueError
        matches = re.match(r"^([^;]+)(; length=([0-9]+))?$", header,
                           re.IGNORECASE)
        header_mtime = parse_http_date(matches.group(1))
        header_len = matches.group(3)
        if header_len and int(header_len) != size:
            raise ValueError
        if mtime > header_mtime:
            raise ValueError
    except (AttributeError, ValueError, OverflowError):
        return True
    return False
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBUHl0aG9uRGphbmdvJTNBJTNBR29waW5hdGhyZWRkeTY2" repo-name="PythonDjango"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
