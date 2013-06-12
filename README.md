
To install
==========

Make sure you're running Emacs 23+

Put ``flymake-cursor.el`` (in this repository) in your site-lisp directory (or add it manually)

Put ``pylint_etc_wrapper.py`` somewhere and modify the path below to match.

Put the following in your ``.emacs``:

```
(setq pycodechecker "/path/to/pylint_etc_wrapper.py")
(when (load "flymake" t)
  (load-library "flymake-cursor")
  (defun dss/flymake-pycodecheck-init ()
    (let* ((temp-file (flymake-init-create-temp-buffer-copy
                       'flymake-create-temp-inplace))
           (local-file (file-relative-name
                        temp-file
                        (file-name-directory buffer-file-name))))
      (list pycodechecker (list local-file))))
  (add-to-list 'flymake-allowed-file-name-masks
               '("\\.py\\'" dss/flymake-pycodecheck-init)))

(defun dss/pylint-msgid-at-point ()
  (interactive)
  (let (msgid
        (line-no (line-number-at-pos)))
    (dolist (elem flymake-err-info msgid)
      (if (eq (car elem) line-no)
            (let ((err (car (second elem))))
              (setq msgid (second (split-string (flymake-ler-text err)))))))))

(defun dss/pylint-silence (msgid)
  "Add a special pylint comment to silence a particular warning."
  (interactive (list (read-from-minibuffer "msgid: " (dss/pylint-msgid-at-point))))
  (save-excursion
    (comment-dwim nil)
    (if (looking-at "pylint:")
        (progn (end-of-line)
               (insert ","))
        (insert "pylint: disable-msg="))
    (insert msgid)))

(setq auto-mode-alist (append (list (cons "\\.py\\'" 'python-mode))
                              auto-mode-alist))

(add-hook 'python-mode-hook 'flymake-mode)


````

In each of your Python virtualenv environments, do:

* ``pip install pylint``
* ``pip install pyflakes``

Then run emacs from inside that ve.

