**Poor Man's Git**

```emacs-lisp

(defun tools/git-status () (interactive) (shell-command "git status" "*git*"))

(defun tools/remove-output-async ()
  (add-to-list 'display-buffer-alist
			   (cons "\\*Async Shell Command\\*.*" (cons #'display-buffer-no-window nil))))

(defun tools/git-add ()
  (interactive)
  (let (which-file) (setq which-file (read-file-name "git add ~"))
	   (async-shell-command (concat "git add " which-file))))

(defun tools/git-commit ()
  (interactive)
  (let (msg)
	(setq msg (read-string "git commit -m "))
	(shell-command (concat "git commit -m '" msg "'") "*git*")))

(defun tools/git-push (origin branch)
  (interactive)
  (shell-command (concat "git push -u " origin " " branch)))

(defun tools/git-remote-add (origin url)
  (interactive)
  (shell-command (concat "git remote add " origin " " url)))

(defun tools/git-remote-add-origin ()
  (interactive)
  (let (url)
	(setq url (read-string "url"))
	(tools/git-remote-add "origin" url)))

(defun tools/git-push-origin ()
  (interactive)
  (let (branch b)
	(setq branch (read-string "master?"))
	(if (or (not branch) (equal branch "") (equal branch " "))
		(setq b "master")
	  (setq b branch))
	(tools/git-push "origin" b)))





```
