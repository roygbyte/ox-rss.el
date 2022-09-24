# ox-rss.el

My personal copy of Bastien Guerry's ox-rss library. Includes some small adjustments to suit my particular use case. Tested for Emacs 28.1.

## Org-roam file excerpts

Wouldn't it be nice if articless in your RSS feed included excerpts that were automatically generated? Just add `:EXCERPT_FROM_ID:` as an entry property, and set the property value to the ID of the org-roam file you'd like to extract the excerpt from. The first paragraph element in the corresponding file will be included as the RSS entry's description.

``` org
* On comics and code
:PROPERTIES:
:PUBDATE: <2022-08-23 Tue>
:EXCERPT_FROM_ID: e7d501c8-91af-4811-8e24-ec7fe614d3b5
:RSS_PERMALINK: comics_and_code.html
:END:
```

Under the hood, here's what's going on:

``` elisp
   (defun org-file-first-paragraph (id)
     "Given an org roam file by ID, find the first paragraph of the file"
     (if (stringp id)
         (let ((file-path (org-roam-id-to-file-path id)))
           (if (stringp file-path)
               (let ((file-contents (file-contents file-path))
                     (file-tree (org-file-to-element-tree file-path)))
                 (extract-region-from-file file-path (nth 0 (org-tree-to-paragraph-positions file-tree))))
             nil
             ))
       nil))
```
## External links

Perchance you'd like to include the odd link to an article beyond the walls of your humble web domain? Just add `:EXTERNAL: t` as an entry property. During the RSS feed build process, your feed's domain will be omitted from the entry's permalink.

``` org
* Montreal dad makes a CO2 Monitor
:PROPERTIES:
:PUBDATE: <2022-08-23 Tue>
:RSS_PERMALINK: https://makezine.com/article/technology/iot/plan-co2-montreal-dad-makes-a-co2-monitor-for-his-daughters-classroom/
:EXTERNAL: t
:END:
```

Under the hood, here's what's going on:

``` elisp
(defun org-rss-headline (headline contents info)
   ...
   (use-absolute (or (not (org-element-property :EXTERNAL headline))
                 (plist-get info :html-link-use-abs-url)))
   ...
   (publink
       (or (and hl-perm (not use-absolute) (concat hl-perm))
           (and hl-perm (concat (or hl-home hl-pdir) hl-perm))
               (concat
                (or hl-home hl-pdir)
                (file-name-nondirectory
                  (file-name-sans-extension
                  (plist-get info :input-file))) "." htmlext "#" anchor)))
```
