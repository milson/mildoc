This is the documentation generator which compiles the <http://milson.org> documentation pages.

Copying the MILDOC template for your own documentation
===

Create your repository
---

    mkdir my-project
    cd my-project
    git init
    echo "My Project" > README.md
    git add README.md
    git commit -m "initial commit"

Merge the mildoc template into your site
---

    git remote add mildoc git://github.com/milson/mildoc.git
    git fetch mildoc
    git pull mildoc template-only
    # creates `./mildoc`

Updating
---

This will update only the build system, not the templates

    git pull mildoc build-sys-only

Installation
===

  0. Install NodeJS (<http://nodejs.org#download>) and `served`

        npm install -g served

  0. Build the example project

        cd mildoc
        ./build.sh
        cd public
        served 7755

  0. View the built project at <http://localhost:7755>

  0. And when your ready to push to **github pages**

        cd my-project
        git checkout master
        
        
        # MAKE SURE everything is COMMITTED
        # uncommited files will be DELETED


        cd mildoc
        ./build.sh
        mv mildoc/public/ /tmp/my-project_mildoc
        
        git symbolic-ref HEAD refs/heads/gh-pages
        rm .git/index
        git clean -fdx
        mv /tmp/my-project_mildoc/* ./
        
        git add ./
        git commit -m "initial commit"
        git push origin -u gh-pages

        # tag your documentation according to your current software
        git tag docs-v1.0.0
        git push --tags
        
  0. And when you want to **update your github pages**

        # Build the latest docs from master
        cd my-project
        git checkout master
        cd mildoc
        ./build.sh
        rm -rf /tmp/my-project_mildoc
        mv mildoc/public/ /tmp/my-project_mildoc
        
        # Replace the current docs with the latest
        git chekout gh-pages
        git rm -f ./
        mv /tmp/my-project_mildoc/* ./
        git add ./
        git commit -m "updated docs to corresponding v1.1.3 of software"
        
        # update your tag
        git tag docs-v1.1.3
        git push --tags


How it Works
===

  0. You put your header, footer, logo, etc in `src/index.jade` 
  0. You put articles in `src/some-group-name/`, `src/some-other-group-name`, etc
  0. When you run `node build.js`
    * The `*.md` files in `src/some-group-name` are turned into annotated `.html`
      * you can change which languages should be annotated in `config.yml.languages`
    * `config.yml` is auto-updated every time you `node build.js`
      * If `src/my-group/my-doc.md` has YAML front matter, or `src/my-group/my-doc.yml` exsits, the defaults are overridden with that data
    * you can change the order in which groups and documents appear by changing their order in `config.yml`
      * even though it auto-updates, the order you specify will be preserved
    * `src/links.jade` and `src/documents.jade` will be produced from the directory scan
    * `src/index.jade` will be built to contain ALL of the documents

Scans Directories
===

This will scan `src` and group files by directory.

It will find `yml` files for each `md` file (or read the `yaml` "front matter" header, if found).

All updated meta data is written out to `config.yml`, where the order can be adjusted as desired.
However, customizations to `title` and `filename` must be done to the accompanying `.yml` or "front matter".

Process Overview
===

Implemented *
Not Implemented -

  * reads in `config.yml`
  * walks the `src` directory structure
  * for each file
    * if file ends in `.md`
      * if file exists in `config.yml`, load presets
      * if file of same name, but ending in `yml` exsits
        * read in yml
        * replace presets, if any
      * if no presets, use defaults (filename, replace special chars)
      * if not exists in `config.yml`, append to end of list
  - compile jade files
  - remove remnant html
