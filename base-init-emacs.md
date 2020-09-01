
Whats the problem?. we need to generate a markdown files that markdown files itself contain init.el and load all emacs configuration. the configuration we generates also can be re-generate itself in the newer version.

How we do it?. first of all we are storing a configuration in the markdown files. that meaning we need to figure out how we identify the code blocks inside the markdown then we evaluate each block in sequences.

in order to re-generate and create initial of it. first we clone this document to specific folder on our local computer. we provide a way of installing or setting up the init.el in exact step. we go openup this files evaluates commands that generate init.el. we dont do syncing or copying out this files to ~/.emacs.d/init.el instead we generate it.

**1.loading markdown files**

1. provide a path where markdown located
2. create temp buffer insert contents of files
3. got to inital position to get ready searching code-blocks
4. forward each line to the end. 
5. for each of it eval-region

```elisp file-out=~/.emacs.d/init.el tangle=false

(defvar md-block-header "^```elisp")
(defvar md-block-end "^```$")

(defun load-markdown (file-paths &optional evaluator)
  (interactive)
  (when (file-exists-p file-paths)
	(with-temp-buffer
	  (insert-file-contents file-paths)
	  (goto-char (point-min))
	  (while (not (eobp))
		(forward-line 1)
		(let ((starting-pos (progn
							  (re-search-forward md-block-header (point-max) t)
							  (match-end 0)))
			  (end-pos (progn
						 (re-search-forward md-block-end (point-max) t)
						 (match-beginning 0))))
		  (if evaluator
			  (funcall evaluator starting-pos end-pos)
			(eval-region starting-pos end-pos)))))))

;;(load-markdown "~/Desktop/file.md")
```


