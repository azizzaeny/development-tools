
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
	(start-repl repl-name init-script) 
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

(defun send-buffer ())

(defun send-region ()
  (interactive)
  (save-excursion
	(when (and transient-mark-mode mark-active)
	  (send-to-repl (buffer-substring-no-properties (point) (mark)))
	  )
	(setq mark-active nil)))

(defun send-js ())
(defun send-lisp ())
(defun send-python ())
(defun send-markdown-block ())
(defun send-clojurev ())

```

```elisp

(custom-key
  "e" 'send-paragraph
  "l" 'send-line
  "r" 'send-region)

```

fix tabs temporary 

```emacs-lisp
(setq web-mode-markup-indent-offset 2)
_(web-mode-code-indent-offset 2)
(setq indent-tabs-mode nil
    js-indent-level 2)
```
