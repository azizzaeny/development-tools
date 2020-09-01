
**load and Tangle Snippet**


```emacs-lisp

(defun filter-list (@predicate @sequence)
  (delete "e3824ad41f2ec1ed" (mapcar (lambda ($x) (if (funcall @predicate $x)  $x "e3824ad41f2ec1ed" )) @sequence)))


(defun snippet/write-file (list-block)
  (interactive)
  (let ((filename (cdr (assoc 'file list-block)))
		(content  (cdr (assoc 'content list-block))))
	(unless (file-exists-p filename)
	  (let ((dir (file-name-directory filename)))
		(unless (file-exists-p dir)	  
		  (make-directory dir t))))
	(with-temp-buffer
	  (insert content)
	  (write-region (point-min) (point-max) filename))))


(defun snippet/get-file-string (path)
  (with-temp-buffer
	(insert-file-contents path)
	(buffer-string)))


(defun snippet/read-meta (str)
  (interactive)
  (let ((res (list)))
	(with-temp-buffer
	  (insert str)
	  (goto-char (point-min))	  
	  (while (not (eobp))
		(forward-line 1)
	 	(re-search-forward "^```" (point-max) t)
		(let (s e c f)
		  (re-search-forward "file=\\([^\s]+\\)" (point-max) t)
		  (setq s (progn (beginning-of-line) (forward-line 1) (point)))		  
		  (setq f (match-string 1))
		  (re-search-forward "^```" (point-max) t)
		  (setq e (match-beginning 0))
		  (setq c  (buffer-substring-no-properties s e))
		  (add-to-list 'res (list (cons 'start s)
								  (cons 'end e)
								  (cons 'file f)
								  (cons 'content c))))))
	res))

;; todo probably just use seq-filter instead of filter-method above
;; (require 'seq)
;; (seq-filter 'numberp '(1 "2" 3))

(defun snippet/filter-nil-file (list-block)
  (filter-list (lambda (l)
				 (not (equal (cdr (assoc 'file l)) nil))
				 ) list-block))

(defun snippet/tangle-all ()
  (interactive)
  (let ((p (expand-file-name "~/.emacs.d/snippet.md"))
		list-block)
	(setq list-block (snippet/read-meta (snippet/get-file-string p)))
	(mapcar 'snippet/write-file (snippet/filter-nil-file list-block))))


(defun snippet/clean ()
  (interactive)
  (shell-command "rm -rf ~/.emacs.d/snippets/*"))

(defun snippet/reload-yas ()
  (interactive)
  (snippet/clean)
  (snippet/tangle-all)
  (shell-command "cp -rv ./snippets/. ~/.emacs.d/snippets/.")
  (yas-reload-all))

```

**Send Code Block to Repl**

start-repl
change-to-repl
insert-region
call-execute
change-to-window
deactive-mark

```emacs-lisp 

(defun repl/start-repl (&optional repl-name init-script)
  "start a new shell repl"
  (interactive)
  (let ((current-buffer-window (selected-window)))	
	(if (not repl-name)
		(shell "*srepl*")
	  (shell repl-name))	
	(when init-script	  
	  (insert init-script)
	  (sit-for 0.1)
	  (comint-send-input))
	;; go back to origin window
	(select-window current-buffer-window)))

;; (defun repl/start ()
;;   (interactive)
;;   (repl/start-repl))

;; (defun repl/stop-repl ()
;;   "stop buffer repl"
;;   (interactive)
;;   ;; todo check if existsted buffer
;;   (kill-buffer "*srepl*"))

(defun repl/send-to-repl (input-str)
  (interactive)
  (let ((current-window (selected-window)))
	;; todo: check if exists buffer
	(switch-to-buffer-other-window "*srepl*")
	(goto-char (point-max))
	(insert input-str)
	(comint-send-input)
	(select-window current-window)))

(defun repl/send-buffer ()
  (interactive)
  (repl/send-to-repl
   (buffer-substring-no-properties (point-min) (point-max))))

(defun repl/send-line ()
  (interactive)
  ;; todo save-excursion
  (let ((init-point (point))) ;; save initial potitions
	(beginning-of-line)
	(set-mark (point)) ;; creating marker from begining of the line
	(end-of-line) ;; to the edge line
	(repl/send-to-repl
	 (buffer-substring-no-properties (point) (mark)))
	(setq mark-active nil)
	(goto-char init-point)))

(defun repl/send-region ()
  "send region when mark is active"
  (interactive)
  (save-excursion
	(if (and transient-mark-mode mark-active)
		(repl/send-to-repl
		 (buffer-substring-no-properties (point) (mark))))
	)
  (setq mark-active nil))

(defun repl/send-region-or-line ()
  (interactive)
  (save-excursion
	(repl/start-repl)
	(if (and transient-mark-mode mark-active)
		(repl/send-region)
	  (repl/send-line)
	  )
	(setq mark-active nil)))

(defun repl/send-markdown-block ())
(defun repl/send-markdown-buffer())

;; simple but works
;; todo fix: behaviour when no defun 
(defun repl/send-lisp ()
  (interactive)  
  (set-mark (line-beginning-position))
  (forward-sexp)
  (repl/send-to-repl
   (buffer-substring-no-properties (point) (mark)))
  (setq mark-active nil)
  (forward-sexp)) ;; go to the next expression

(defun repl/send-clojure ())
;; todo clojure forward send-expression


;; this is also simple but works
;; from atlantis.net
;; todo look at js2 implementation
(require 'js2-mode)
(defun repl/send-javascript ()
  (interactive)
  (let ((fn (js2-mode-function-at-point (point))))
	(when fn
	  (let ((beg (js2-node-abs-pos fn))
			(end (js2-node-abs-end fn)))
		(repl/send-to-repl
		 (buffer-substring-no-properties beg end))
		))))

;; (srepl/stop-repl)
;; (srepl/start-repl "clojure")
;; (srepl/start-repl "ls" "**foo**")
;; (srepl/send-to-repl "clojure")

(custom-key
  "r e"  'repl/send-line
  "r b"  'repl/send-buffer
  "r j"  'repl/send-javascript
  "r r"  'repl/send-region
  "r p"  'repl/stop-repl
  "e"      'repl/send-region-or-line
  "r s"  'repl/start)

```
