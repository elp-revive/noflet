[![JCS-ELPA](https://raw.githubusercontent.com/jcs-emacs/badges/master/elpa/v/noflet.svg)](https://jcs-emacs.github.io/jcs-elpa/#/noflet)

# noflet - Local function decoration

[![CI](https://github.com/elp-revive/noflet/actions/workflows/test.yml/badge.svg)](https://github.com/elp-revive/noflet/actions/workflows/test.yml)

`noflet` is dynamic, local, advice for Emacs-Lisp code.

`noflet` also has an Emacs indentation function for `flet`
like macros.

It's great for test code when you need to mock another function.

This is useful for defining functions that override a base definition
in some way. You get access to the original (before you re-defined it)
function through a different name.

Use it like this:

```elisp
(noflet ((find-file-noselect (file-name)
           (if (string-match-p "^#.*" file-name)
               (this-fn "/tmp/mytest")
               (this-fn file-name)))
         (expand-file-name (file-name &optional thing)
           (if (string-match-p "^#.*" file-name)
               (concat "/tmp" (file-name-as-directory 
                               (substring file-name 1)))
               (funcall this-fn file-name thing))))
  (expand-file-name "#/thing"))
```

This specifies that two functions should be overridden:

* `find-file-noselect` is changed so that if the file-name begins with `#` a different file-name altogether is opened
* `expand-file-name` is changed so that if the file-name begins with `#` it's resolved via `/tmp`

In both cases `this-fn` is used to access the original function
definition of these common Emacs functions.

## Decorating results

`noflet` can also be used to decorate results, just like `around-advice`:

```elisp
(noflet ((find-file (file-name &optional wildcards)
           (let ((result (funcall this-fn file-name wildcards)))
              (with-current-buffer result
                 (setq some-buffer-local "special"))
              result)))
    (with-current-buffer (find-file "~/some-file")
      (message "buffer local var is: %s" some-buffer-local)))
```

This overrides `find-file` to set a local variable. There are
surely better ways to do it than this but it illustrates the point.


## Lexical version

Because we include a good indenting function we also include a lexical
`flet`. It's just a wrapper for `cl-flet`.
