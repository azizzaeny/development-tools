
What is connected programming?. in short term it is, the programming without a structure, connect to your creation mode, eval in repl, literate programming, immediete feedback loop,   

this is long story problem that the partial solutions is actually out there and implementation are scatered arounds in context specified problems of their own. my intention is to entangle, to breakdown each part, to take what we need and dispose what we dont need.   


```elisp

(defvar active-repl "")

(defun start-repl (&optional repl-name init-script)
  (interactive)
  (let ((current-buffer-window (selected-window)))
	(if (not repl-name)
		(setq active-repl "*primary-repl*")
	  (setq active-repl repl-name))
	(progn
	  (shell active-repl)
	  (when init-script
		(insert init-script)
		(sit-for 0.1)
		(comint-send-input))
	  (select-window current-buffer-window))))

(defun send-to-repl (input-str &optional repl-name init-script)
  (interactive)
  (let ((current-window (selected-window)))
	;; (start-repl repl-name init-script) 
	(switch-to-buffer-other-window active-repl)
	(goto-char (point-max))
	(insert input-str)
	(comint-send-input)
	(select-window current-window)))

;; higher order functions to make it looks inteligence
;; by detecting automaticly what type of code to sends where begining statement and what common itended block code to send

(defun send-line ()
  (interactive)
  (save-excursion
	(let ((init-p (point)))
	  (beginning-of-line)
	  (set-mark (point))
	  (end-of-line)
	  (send-to-repl (buffer-substring-no-properties (point) (mark)))
	  (setq mark-active nil)
	  (goto-char init-p))))

(defun send-paragraph ()
  (interactive)
  (save-excursion
	(let ((init-p (point)))
	  (re-search-backward "\n[\t\n ]*\n+" nil t)
	  (skip-chars-backward "\n\t ")
	  (forward-char)
	  (send-to-repl (buffer-substring-no-properties (point) init-p))
	  )))

(defun send-buffer ()
  (interactive)
  (send-to-repl (buffer-substring-no-properties (point-min) (point-max))))


(defun send-region ()
  (interactive)
  (save-excursion
	(when (and transient-mark-mode mark-active)
	  (send-to-repl (buffer-substring-no-properties (point) (mark)))
	  )
	(setq mark-active nil)))

(defun send-js ())

(defun send-lisp ()
  (interactive)
  (set-mark (line-beginning-position))
  (forward-sexp)
  (send-to-repl (buffer-substring-no-properties (point) (mark)))
  (setq mark-active nil)
  (forward-sexp))

(defun send-python ())
(defun send-markdown-block ())
(defun send-clojurev ())


(defun copy-connectedp-to-emacs ()
  (interactive)
  (save-current-buffer)
  (unless (equal (buffer-name) "connected-program.md")
	(copy-file (buffer-file-name) (expand-file-name "~/.emacs.d/") t)))

;;(copy-connectedp-to-emacs)
;; (load-markdown "~/.emacs.d/connected-program.md")
```

Tangle markdown block out

```elisp 
;; note: required base-init-emacs functions.

(defvar md-file-ref ":file=\\([^\s+]+\\)")

(defun tangle-current-block ()
  (interactive)
  (save-excursion
    (let ((starting-pos (progn (re-search-backward "^```" (point-min) t) (match-end 0)))    
          (end-pos (progn (re-search-forward md-block-end (point-max) t) (match-beginning 0))))
      (let ((file-ref (or (progn (re-search-backward md-file-ref starting-pos t) (match-string 1)) nil))
            (start-content (progn (goto-char starting-pos) (beginning-of-line) (forward-line 1) (point))))
        (when file-ref
          (write-to-file file-ref (buffer-substring-no-properties start-content end-pos)))
        ))))


(defun tangle-buffer ())
(defun tangle-current-buffer ())

```

Hotkey - bind

```elisp

(custom-key
  "e" 'send-paragraph
  "l" 'send-line
  "r" 'send-region
  "b" 'send-buffer
  "t" 'tangle-current-block)

```
