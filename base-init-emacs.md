
Whats the problem?. we need to generate a markdown files that markdown files itself contain init.el and load all emacs configuration. the configuration we generates also can be re-generate itself in the newer version.

How we do it?. first of all we are storing a configuration in the markdown files. that meaning we need to figure out how we identify the code blocks inside the markdown then we evaluate each block in sequences.

in order to re-generate and create initial of it. first we clone this document to specific folder on our local computer. we provide a way of installing or setting up the init.el in exact step. we go openup this files evaluates commands that generate init.el. we dont do syncing or copying out this files to ~/.emacs.d/init.el instead we generate it.  

**1.loading markdown files**  

1. provide a path where markdown located
2. create temp buffer insert contents of files
3. got to inital position to get ready searching code-blocks
4. forward each line to the end. 
5. for each of it eval-region

```emacs-lisp 

(defvar md-block-header "^```elisp")
(defvar md-block-end "^```$")

(defun load-markdown (file-paths &optional evaluator)  
  (interactive)  
  (when (file-exists-p (expand-file-name file-paths))	
	(with-temp-buffer	  
	  (insert-file-contents file-paths)
	  (goto-char (point-min))
	  (while (not (eobp))
		(forward-line 1)
		(let ((starting-pos (progn (re-search-forward md-block-header (point-max) t)
								   (match-end 0)))
			  (end-pos (progn (re-search-forward md-block-end (point-max) t)
							  (match-beginning 0))))
		  (if evaluator
			  (funcall evaluator starting-pos end-pos)
			(eval-region starting-pos end-pos)))))))

(defvar md-base-init "~/.emacs.d/base-init-emacs.md")
;; todo : fix load all markdown in the directory
(load-markdown md-base-init)

(custom-set-variables
 '(custom-safe-themes t))

```

The above code is not executed when initial startup of emacs because we are looking code-blocks of elisp not emacs-lisp (later on we can use meta data of code blocks instead using this methods).
we use above code as a content to generate our init.el and those init.el will load all base-init-emacs that is this files. sounds like paradoks ;)

**Generating init.el**  

To generate init.el we would use this base init emacs as inital starting pos. generate-init-el only invoked once, when we starting first time to used this configuration.

```elisp

(defvar md-block-header "^```elisp")
(defvar md-block-end "^```$")

(defun load-markdown (file-paths &optional evaluator)  
  (interactive)  
  (when (file-exists-p (expand-file-name file-paths))	
	(with-temp-buffer	  
	  (insert-file-contents file-paths)
	  (goto-char (point-min))
	  (while (not (eobp))
		(forward-line 1)
		(let ((starting-pos (progn (re-search-forward md-block-header (point-max) t)
								   (match-end 0)))
			  (end-pos (progn (re-search-forward md-block-end (point-max) t)
							  (match-beginning 0))))
		  (if evaluator
			  (funcall evaluator starting-pos end-pos)
			(eval-region starting-pos end-pos)))))))

(defvar md-base-init "~/.emacs.d/base-init-emacs.md")

(defun write-to-file (file-location contents)
  (interactive)
  (unless (file-exists-p file-location)
	(let ((dir (file-name-directory file-location)))
	  (unless (file-exists-p dir)
		(make-directory dir t))))
  (with-temp-buffer
	(insert contents)
	(write-region (point-min) (point-max) file-location)))


(defun search-first-emacs-lisp ()
  (interactive)
  (let ((contents (buffer-substring-no-properties (point-min) (point-max))))
	(with-temp-buffer
	  (insert contents)
	  (goto-char (point-min))
	  (let (( s (progn (re-search-forward "^```emacs-lisp" (point-max) t) (match-end 0) (forward-line 1) (point)))
			( e (progn (re-search-forward "^```$" (point-max) t) (match-beginning 0))))
		(buffer-substring-no-properties s e)))))

;; (search-first-emacs-lisp) 1093 1836

(defvar init-el-location  "~/.emacs.d/init.el")

(defun generate-init-el ()
  (interactive)
  (let ((source (search-first-emacs-lisp))
		(location init-el-location))
	(write-to-file location source)))

;; (generate-init-el)

```

**Upsync init.el and markdown configuration**  

in order to synchronize configuration you need to visit base-init in your buffer first then hit update-reload-init-el.

```elisp

;; Note: this would cause and unknown behaviour: copy-file to emacs.d directory when not the specified buffer visied

(defun update-reload-init-el ()
  (interactive)
  (save-current-buffer)
  (generate-init-el)
  (unless (equal (buffer-name) "base-init-emacs.md")
	(copy-file (buffer-file-name) (expand-file-name "~/.emacs.d/") t)))

```


**General & Common Emacs Configuration**  

bellow is the basic emacs setups.

```elisp

;; Basic UI

(scroll-bar-mode -1)
(tool-bar-mode   -1)
(tooltip-mode    -1)
(menu-bar-mode   -1)
(set-language-environment "UTF-8")
(prefer-coding-system 'utf-8)
(setq use-dialog-box nil)
(setq inhibit-startup-message t)
(setq inhibit-startup-echo-area t)
(setq initial-scratch-message nil)
(setq ring-bel-function 'ignore)
(setq visible-bell t)
(setq custom-safe-themes t)
(setq ns-use-proxy-icon  nil)
(setq frame-title-format nil)
(setq find-file-visit-truename t)
(setq large-file-warning-threshold (* 25 1024 1024))
(setq comment-style 'extra-line)
(fset 'yes-or-no-p 'y-or-n-p)
(setq redisplay-dont-pause t)

;; Backup files 

(setq make-backup-files nil)
(setq auto-save-default nil)
(setq create-lockfiles nil)
(setq backup-inhibited t)
(setq auto-save-list-file-prefix nil)

;; line number and spacing

(setq line-number-mode nil)
(setq indicate-empty-lines t)
(setq global-hl-line-mode nil)
(setq tab-width 2)
(setq toggle-truncate-lines t)
(setq indent-tabs-mode nil)

;; Cursor, mouse-wheel, scroll

(set-default (quote cursor-type) t)
(blink-cursor-mode -1)
(defvar cursor-initial-color (face-attribute 'cursor :background))
(setq mouse-wheel-scroll-amount '(1 ((shift) . 1)))  ;; one line at a time
(setq mouse-wheel-progressive-speed nil) ;; don't accelerate scrolling
(setq mouse-wheel-follow-mouse 't) ;; scroll window under mouse
(setq scroll-step 2)  ;; keyboard scroll one line at a time
(setq scroll-conservatively 10000)
(setq scroll-preserve-screen-position t)

;; copy paste from and to x clipboard

;; (defun copy-to-clipboardx (&str)
;;   (interactive)
;;   (shell-command-to-string "xclip -selection clipboard"))


;; (setq interprogram-cut-function nil)

```


**Third Party Package**   
initialization third party package with el-get bundler

```elisp  

(defun initialize-el-get ()
  (add-to-list 'load-path "~/.emacs.d/el-get/el-get")  
  (unless (require 'el-get nil 'noerror)
	(with-current-buffer
		(url-retrieve-synchronously
		 "https://raw.githubusercontent.com/dimitri/el-get/master/el-get-install.el")
	  (goto-char (point-max))
	  (eval-print-last-sexp)))
  (add-to-list 'el-get-recipe-path "~/.emacs.d/el-get-user/recipes"))

(initialize-el-get)

;; (el-get-bundle github-theme
;;   :url "https://raw.githubusercontent.com/dakrone/emacs-github-theme/master/github-theme.el")

(el-get-bundle github-theme
  :url "https://raw.githubusercontent.com/philiparvidsson/GitHub-Theme-for-Emacs/master/github-theme.el")

(el-get-bundle polymode/polymode
  :type github :pkgname "polymode/polymode")

(el-get-bundle polymode/poly-markdown
  :type github :pkgname "polymode/poly-markdown")

(el-get-bundle defunkt/markdown-mode
  :type github :pkgname "defunkt/markdown-mode")

(el-get-bundle mooz/js2-mode
  :type github :pkgname "mooz/js2-mode")

(el-get-bundle clojure-mode)
(el-get-bundle parinfer)
(el-get-bundle paredit)
(el-get-bundle rainbow-delimiters)
(el-get-bundle aggressive-indent)
(el-get-bundle smartparens)

(el-get-bundle fgallina/multi-web-mode
  :type github :pkgname "fgallina/multi-web-mode")

(el-get-bundle emmet-mode)
(el-get-bundle python-mode)
(el-get-bundle yasnippet)
(el-get-bundle gist)
(el-get-bundle counsel)
(el-get-bundle ivy)
(el-get-bundle which-key)
(el-get-bundle auto-complete)
(el-get-bundle multiple-cursors)
(el-get-bundle autopair)

(el-get-bundle iqbalansari/restart-emacs
  :type github :pkgname "iqbalansari/restart-emacs")

(el-get-bundle general)

(el-get-bundle typescript)
(el-get-bundle tide)

(el-get-bundle php-mode)

(el-get-bundle xclip)
(el-get-bundle restclient)
(el-get 'sync)

(package-initialize)
```


**Configuration of 3rd Party Package**   

```elisp
;; ah.. yes. white themes
(load "~/.emacs.d/el-get/github-theme/github-theme.el")
(load-theme 'github t)
;;(load-theme 'github-modern t)



;; uniqify buffer
(require 'uniquify)
(setq uniquify-buffer-name-style 'post-forward-angle-brackets)


;; show-paren
(require 'paren)
(setq show-paren-delay 0.4)
(set-face-foreground 'show-paren-match "#def")
(set-face-attribute 'show-paren-match nil :weight 'extra-bold)
(show-paren-mode 1)


;; markdown-mode
(require 'markdown-mode)
(with-eval-after-load 'markdown-mode
  (autoload 'gfm-mode "markdown-mode"
    "Major mode for editing GitHub Flavored Markdown files" t)
  (add-to-list 'auto-mode-alist '("README\\.md\\'" . gfm-mode)))


;; polymode poly-markdown
(require 'polymode)
(require 'poly-markdown)
(with-eval-after-load 'poly-markdown
  (add-to-list 'auto-mode-alist '("\\.md" . poly-markdown-mode)))

;; restclient
(require 'restclient)

;; js2-mode

;; clojure-mode
(require 'clojure-mode)
(setq clojure-indent-style 'always-indent)
(setq comment-column 0)


;;parinfer
(require 'parinfer)
(setq parinfer-extensions
	  '(defaults pretty-parens paredit smart-tab smart-yank))

;; (add-hook 'clojure-mode-hook #'parinfer-mode)
;; (add-hook 'emacs-lisp-mode-hook #'parinfer-mode)
;; (add-hook 'common-lisp-mode-hook #'parinfer-mode)
;; (add-hook 'scheme-mode-hook #'parinfer-mode)
;; (add-hook 'lisp-mode-hook #'parinfer-mode)

(setq parinfer-auto-switch-indent-mode nil)  ;; default
(setq parinfer-auto-switch-indent-mode-when-closing nil)  ;; default
(setq parinfer-delay-invoke-threshold 6000)  ;; default
(setq parinfer-delay-invoke-idle 0.3)  ;; default


;; paredit

;; rainbow-delimiters
(require 'rainbow-delimiters)
(add-hook 'clojure-mode-hook #'aggressive-indent-mode)


;; smartparen
(require 'smartparens-config)
(add-hook 'clojure-mode-hook #'smartparens-mode)
(add-hook 'js-mode-hook #'smartparens-mode)


;;aggressive indent
(require 'aggressive-indent)
(add-hook 'emacs-lisp-mode-hook #'aggressive-indent-mode)
(add-hook 'css-mode-hook #'aggressive-indent-mode)
(add-hook 'clojure-mode-hook #'aggressive-indent-mode)
(global-aggressive-indent-mode 1)
(add-to-list 'aggressive-indent-excluded-modes 'html-mode)


;;multi-web
(require 'multi-web-mode)
(setq mweb-default-major-mode 'html-mode)
(setq mweb-tags '((php-mode "<\\?php\\|<\\? \\|<\\?=" "\\?>")
                  (js-mode "<script +\\(type=\"text/javascript\"\\|language=\"javascript\"\\)[^>]*>" "</script>")
                  (css-mode "<style +type=\"text/css\"[^>]*>" "</style>")))
(setq mweb-filename-extensions '("php" "htm" "html" "ctp" "phtml" "php4" "php5"))
(multi-web-global-mode 1)


;;emmet
(require 'emmet-mode)
;;(add-hook 'css-mode-hook  'emmet-mode) 
;;(add-hook 'html-mode-hook 'emmet-mode)
(setq emmet-indentation 2)
(setq emmet-self-closing-tag-style " /")


;; python
;; yasnippet
(require 'yasnippet)
(yas-global-mode 1)
(setq yas-snippet-dirs '("~/.emacs.d/snippets/"))


;; gist
;; git config --global github.user <your-github-user-name>
;; git config --global github.oauth-token <your-personal-access-token-with-gist-scope>

;; counsel, ivy, swiper
(require 'counsel)
(require 'ivy)
(require 'swiper)
(ivy-mode 1)
(setq ivy-use-virtual-buffers t)
(setq enable-recursive-minibuffers nil)
;; (setq search-default-mode #'char-fold-to-regexp)
(setq ivy-initial-inputs-alist nil)
(setq ivy-count-format "")
(setq ivy-display-style nil)
(setq ivy-minibuffer-faces nil)
(add-to-list 'ivy-highlight-functions-alist
             '(swiper--re-builder . ivy--highlight-ignore-order))

(setq ivy-re-builders-alist
	  '((ivy-switch-buffer . ivy--regex-plus)
		(swiper . ivy--regex-plus)))



;; which-key
(require 'which-key)
(which-key-mode)
(which-key-setup-side-window-bottom)
(which-key-setup-minibuffer)
(setq which-key-idle-delay 1)
(setq which-key-idle-secondary-delay 0.01)
(setq which-key-popup-type 'minibuffer)


;; auto-complete
;; (require 'auto-complete)
;; (ac-config-default)


;; multi-cursors
(require 'multiple-cursors)

;; auto-pair
(require 'autopair)
;;(autopair-global-mode)

;; tide
(defun setup-tide-mode ()
  (interactive)
  (tide-setup)
  (flycheck-mode +1)
  (setq flycheck-check-syntax-automatically '(save mode-enabled))
  (eldoc-mode +1)
  (tide-hl-identifier-mode +1)
  ;; company is an optional dependency. You have to
  ;; install it separately via package-install
  ;; `M-x package-install [ret] company`
  (company-mode +1))


;; aligns annotation to the right hand side
(setq company-tooltip-align-annotations t)

;; formats the buffer before saving
;; (add-hook 'before-save-hook 'tide-format-before-save)
;; (add-hook 'typescript-mode-hook #'setup-tide-mode)

;;(require 'ts-comint)
;; run-ts

;; xclip
(require 'xclip)
(turn-on-xclip)


;; Custom Key Bindings Basic Editing/Searching

(defvar custom-bindings-map (make-keymap)
  "A keymap for custom bindings.")

(general-define-key
 "C-g"     'minibuffer-keyboard-quit
 "C-s"     'counsel-grep-or-swiper
 "C-x C-f" 'counsel-find-file
 "C-x ag"  'counsel-ag
 "C-x f"   'counsel-describe-function
 "C-x l"   'counsel-find-library
 "C-x f" nil
 "C-x <left>" nil
 "C-x <right>" nil
 )

(defconst custom-key "C-c")

(general-create-definer
  custom-key :prefix "C-c")

(define-minor-mode custom-bindings-mode
  "A mode that activates custom-bindings."
  t nil custom-bindings-map)

(custom-key
  "m e" 'mc/edit-lines
  "m n" 'mc/mark-next-lines
  "m e" 'emmet-expand-line
  "<right>" 'shrink-window-horizontally
  "<left>"  'enlarge-window-horizontally
  "<down>"  'shrink-window
  "<up>"    'enlarge-window)

(custom-bindings-mode 1)

(custom-key
  "s ol" 'sort-lines
  "s op" 'sort-paragraphs
  "s oc" 'sort-columns
  "s or" 'reverse-region )


;; (remove-output-async) ;; make it silence

;; (custom-key
;;   "g s" 'git/status  
;;   "g a" 'git/add
;;   "g c" 'git/commit
;;   "g m" 'git/commit  
;;   "g p" 'git/push-origin
;;   "g u" 'git/push-origin
;;   "g r" 'git/remote-add-origin)


;; (custom-key
;;   "s nr" 'snippet/reload-yas
;;   "s nc" 'snippet/clean
;;   "s nt" 'snippet/tangle-all)

;; (defun emacs/sync-dotfile ()
;;   (interactive)
;;   (shell-command "./bin/sync"))

;; (defun emacs/reload-markdown-init ()
;;   (interactive)
;;   (load-markdown "~/.emacs.d/readme.md"))

;; (custom-key
;;   "sy" 'emacs/sync-dotfile
;;   "rl" 'emacs/reload-markdown-init
;;   "rs" 'restart-emacs)

;;(setq typescript-indent-level
;;	  (or (plist-get (tide-tsfmt-options) ':indentSize) 2))

(setq web-mode-markup-indent-offset 2)

(setq indent-tabs-mode nil
    js-indent-level 2)

(setq typescript-indent-level 2)

```
