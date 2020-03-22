Impatient Mode
==============

See the effect of your HTML as you type it.

 * [YouTube demo](http://youtu.be/QV6XVyXjBO8)

Installation through MELPA
--------------------------

The easiest way to get up and running with _impatient-mode_ is to
install it through [MELPA](http://melpa.milkbox.net/). If you're not
already using MELPA,
[it's quite easy to setup.](http://melpa.milkbox.net/#installing)

Installation from Source
------------------------

If you are installing from source, please note that this package
requires both _simple-httpd_ and _htmlize_ in order to operate. The
_simple-httpd_ webserver runs within emacs to serve up your buffers as
you edit them. _htmlize_ is used to send font lock highlighting to
clients for non-HTML buffers.

_simple-httpd_ can be installed through MELPA or directly from GitHub.

 * http://melpa.milkbox.net/
 * https://github.com/skeeto/emacs-http-server

_htmlize_ is also available through MELPA.

Once you have installed _simple-httpd_ and _htmlize_ and you've cloned
_impatient-mode_, you can add _impatient-mode_ to your load path and
require it:

```el
(add-to-list 'load-path "~/.emacs.d/impatient-mode")
(require 'impatient-mode)
```

Using _impatient-mode_
----------------------

Enable the web server provided by _simple-httpd_:

```el
M-x httpd-start
```

Publish buffers by enabling the minor mode `impatient-mode`.

```
M-x impatient-mode
```

And then point your browser to http://localhost:8080/imp/, select a
buffer, and watch your changes appear as you type!

If you are editing HTML that references resources in other files (like
CSS) you can enable impatient-mode on those buffers as well. This will
cause your browser to live refresh the page when you edit a referenced
resource.

Except for `html-mode` buffers, buffer contents will be run through
a user-defined filter. The default user filter is `htmlize`, but you can set your own with `imp-set-user-filter`. The user filter is nothing but a regular elisp function. Here's how you would define a basic filter:

```el
(defun my-filter (_)
  (princ "<html><body>test</body></html>" (current-buffer)))
```

The original editing buffer is passed along the user filter as a parameter, which we didn't use in the previous example, but which is demonstrated in the following example:

```el
(defun my-filter (buffer)
  (let ((count 
     (with-current-buffer buffer
           (count-words-region (point-min) (point-max)))))
    (princ (format "<html><body>%d</body></html>" count) (current-buffer))))
```

You can remove user filters with `imp-remove-user-filter`, which will reset the default `htmlize`. For reference, this is how the default user function is defined:

```el
(defun default-user-filter (buffer)
  "Htmlization of buffers before sending to clients."
  (let ((html-buffer (save-match-data (htmlize-buffer buffer))))
    (princ (with-current-buffer html-buffer (buffer-string)))
    (kill-buffer html-buffer)))
```

Security implications
---------------------

Please be aware that enabling `impatient-mode` exposes the whole directory in which the file resides, not only the file itself. If our file is accessible under `http://localhost:8080/imp/live/example.txt/`, it is possible to access `http://localhost:8080/imp/live/example.txt/a-file-in-the-same-directory/or-even/a-subdirectory-of-it.txt`. It's especially dangerous when enabling `impatient-mode` for files like `~/.bashrc` because it allows to access any file in the user's home directory files such as `~/.ssh/id_rsa`.

This behavior is not a bug, it is needed for the HTML files to work properly along with their resources (such as CSS and JS). Please be aware of what is exposed and/or configure your filewall accordingly.
