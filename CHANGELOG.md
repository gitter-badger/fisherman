# Change Log

+ [0.4.0](#040) :gem:
* [0.3.1](#031)
* [0.3.0](#030)
* [0.2.0](#020)
* [0.1.0](#010)


## [0.4.0][v040] - 2016-01-11

:anchor: Introducting Fisherman's official website :construction:

<a href="http://fisherman.sh">
<img src="https://cloud.githubusercontent.com/assets/8317250/12229311/c0eea838-b889-11e5-94eb-280d95fbdd49.png">
</a>

. Powered by Jekyll and hosted by GitHub pages.

* Refactor `fisher install` / `fisher uninstall` by extracting the logic to enable / disable plugins into `__fisher_plugin`. The algorithm to enable/disable plugins is essentially the same. The only difference is _enable_, copies/symlinks files and disable removes them from `$fisher_config/...`. Closes #45.

* Add support for legacy oh-my-fish! plugins using `.load` initialization files. Closes #35.

* Add support for [Tackle](https://github.com/justinmayer/tackle) Fish framework initialization modules. Closes #35.

* :gem: Add support for plugins that share scripts in languages like Python or Perl. For example `oh-my-fish/plugin-vi-mode` assumes there is a `vi-mode-impl.py` file in the same path of the running script. This opens the door for including code snippets in other languages.

    * Any files inside a `share` directory, except for `*.md` or `*.fish` files, are copied to `$fisher_config/functions`. This allows you to run legacy plugins that retrieve the currently running script path with `(dirname (status -f))` out of the box.

    * A cleaner alternative is using the new `$fisher_share` variable like this: `python $fisher_share/my_plugin_script.py`.

    * `$fisher_share` points to `$fisher_config/share` by default, but you may change this in your user `config.fish`. This path contains copies (or symbolic links) to the same script files copied to `$fisher_config/functions`.

    * Introduce the `$fisher_share_extensions` variable to let you customize what extensions Fisherman is aware of. Only extensions in this array will be processed during the install process. The default is `py rb php pl awk sed`.

    * `.fish` and `.md` extensions are always ignored.


* Remove ad-hoc debug `d` function created by mistake in the Fisherman config.fish file. Closes #34.

* Remove almost useless `fisher --alias`. You can still create aliases using `$fisher_alias`. It's difficult to add auto-complete to this feature, and even if we do so, it is slow.

* Fix bug introduced in the previous release caused by swapping the lines that calculate the index of the current plugin being installed/updated/uninstalled and the line that displays the value, causing the CLI to show incorrect values. Closes #36. Thanks @kballard

* Add `cache`, `enabled` and `disabled` options to `fisher --list`. Now you can type `fisher -l enabled` to get a list of what plugins are currently enabled.

* Add new `$fisher_plugins` universal variable to keep track of what plugins are enabled / disabled.

* Update completions after a plugin is installed, updated or uninstalled.

* Improve autocomplete speed by removing the descriptions from plugins installed with custom URLs.

* `fisher --list` displays nothing and returns 1 when there are no plugins installed. Closes #38.

* `fisher uninstall` does not attempt to uninstall plugins already disabled by looking at the `$fisher_plugins` array. `--force` will bypass this. Closes #40

## [0.3.1][v031] - 2016-01-10

> This patch contains several amends for 0.3.0 and other minor documentation corrections.

* Major documentation revision and rewrite.

* `fisher help` shows `fisher(1)` by default now.

* Fix a critical bug that was causing `fisher uninstall --force` to remove _not_ the symbolic link, but the actual files. Closes #24

* Rename `orphan` tag to `custom` for plugins installed using a custom URL.

* :warning: Remove `fisher --link` flag and create symbolic links by default for local paths. The user does not have to worry about symbolic links or whether the copy is as symbolic link or not anymore. If the user tries to install a local path, then the best thing to do is to create a symbolic link. This also eliminates the need to call update.

* :warning: Remove `fisher --cache` and `fisher --validate`. Now, that these options are separated into their own function and they are intentionally private, there is no need for them.


## [0.3.0][v030] - 2016-01-08

> This release contains several breaking changes a few major improvements. The good news is that the API is starting to look more stable and very unlikely to change drastically again in the future.

### Fixes

* Fix a critical bug in the Makefile that was incorrectly merging any existing user configuration file and the generated Fisherman configuration. Closes #21.

* Fix a bug in install and uninstall that was adding plugin names to fishfiles instead of the URL when interacting with custom URLs. Probably closes #23.

* Fix a bug in install, update and uninstall that was displaying an incorrect plugin count if there was at least on failure.

* Fix bug in `fisher install` that causes install to fail even though it succeeds, due to `wait(1)`'s behavior of returning `1` if there is any output to standard error. Closes #20.

* Fix bug in `fisher uninstall` that was removing plugins from the cache by mistake.

### Add

* Add feature to Makefile to download the index for the first time in order to provide auto-complete before the user can install/update/search, actions which would case the index to be updated.

* Add link to Slack [room](http://fisherman-wharf.herokuapp.com/) in README. Thanks @simnalamburt.

* Add new `$fisher_timeout` configuration variable that lets you specify `curl(1)` `--max-time` option. Without this, `curl` could hang for a long time if you are in a bad connection.

* Add `fisher install --link` to allow installing plugins creating a symbolic link to each of the relevant files to be copied during the install process. If you use ***`--link`*** to install a plugin that is a _path to a directory_ or file, a symbolic link to the directory will be created making local testing more convenient as you are not required to update the plugin's repository to test changes within Fisherman. If you are testing using [Fishtape](https://github.com/fisherman/fishtape) you do not even need to reset the shell session.

* ~~Add `fisher --alias[=<command>=<alias>]` to simplify creating new aliases for `fisher` commands. Use `fisher --alias` without arguments to list the current set of aliases. Also add auto-complete for aliases to install, update or uninstall. Note that aliases are **not** persisted this way. To save your aliases use `$fisher_alias` as described in `fisher help config`. Also note that aliases are only auto-complete if you call `fisher --alias`. To auto-complete aliases saved to `$fisher_alias` you can do `fisher --alias (fisher --alias)`.~~

* Add short options for new and old fisher flags:

    * `--file` → `-f`
    * `--list` → `-l`
    * `--alias` → `-a`

### Improvements

* Improve help message for failed installs. Closes ##24. @namandistro

* Improve `fisher --validate` to automatically correct common misspellings, for example when installing a oh-my-fish package, one often types ohmyifsh.

* :point_up: Improve auto-complete performance by extracting the implementation of the different `fisher` flags to `__fisher_*` functions. `completions/fisher.fish` relies heavily in `fisher_search` to query what plugins are available to install/update/uninstall. In this process, numerous calls to `fisher --list` and `fisher --validate`, etc., are made. Now, auto-complete does not have to pay the penalty of entering `fisher`, parsing options, etc. Closes #27. @namandistro

* Improve `fisher --help` output and show up until now poorly documented ***`--list`***, ***`--file`***, etc. flags consistently. Also display available commands after `make install` to improve usability.

* Improve `fisher install` so that it checks whether the plugin you are trying to install, if it is already in the cache, is a symbolic link or not, and installs it as if the `--link` flag was specified.

* Improve `fisher --validate` to retrieve the absolute path to the closest directory of the given items if they are valid local paths. Related #19.

* Improve install to not `git clone` local plugins if a regular `path/to/file` is given to `fisher install`. Instead, copy to the cache using `cp(1)` and if `--link` is used, create a symlink.

* Improve `fisher --validate` to invalidate items with repeated `.` and `-` and allow items that begin with `/` or `./` to support installing plugins from local paths. Related #19.

## :warning: Remove / Rename

* Modify `fisher update` default behavior. Now this command updates Fisherman by default. Use of `--self` and `--me` is also **deprecated**. To read from the standard input use a dash `-`. For example: `fisher --list | fisher update -`. See [Usage of dash in place of a filename](http://unix.stackexchange.com/questions/16357/usage-of-dash-in-place-of-a-filename/16364#16364). Closes #25.

* Rename `--cache` to more descriptive ***`--list`***. Thanks @colstrom.

* Remove `fisher --cache=base` and make it return the base names of all directories in the path by default. To get the full path use printf `printf "$fisher_cache/%s" (fisher --list)`

* Rename undocumented `fisher --translate` flag (again) to `fisher --cache`. This function reads the standard input for a name, URL or local path and calculates the plugin's path relative to the cache. For a name this is simple `$fisher_cache/<name>` for an URL, retrieve the remote URL of every repository until there is a match with the given URL and return the path in the cache of that repository. Finally, if the input is a local path of the form `file:///` it will pass it as is.

* Revert #3. The reason `getopts.fish` was in its own file originally is because @bucaran wanted a standalone, dependency free cli parser solution, arguably slightly faster than having Awk read `getopts.awk` for each use. The performance improvement is negligible at best, but `getopts` is also used by every single command and future commands and plugins are very likely to use it as well, so we might as well use the slightly faster version.


## [0.2.0][v020] - 2016-01-05

* Improve README, added links to screencasts, updated documentation with new changes and fixed other typos and composition errors.

* :warning: Remove `fisher update --cache` in favor of `fisher --list | fisher update` and `fisher uninstall --all` in favor of `fisher --list | fisher uninstall`.

* :warning: Fisherman does not move initialization / configuration files following the convention `name`.config.fish to `$fisher_config/functions`, but to `$fisher_config/conf.d` now and evaluates each `*.config.fish` inside at shell start as usual. Closes #13.

* ~~Add `fisher --cache[=base]` option to retrieve contents in `$fisher_cache`, eliminating flaky usage of `find(1)`~~. Closes #11.

* Fisherman now generates information about plugins installed via custom URLs. For the description, a shortened version of the URL is used. For the URL the full URL is used. For tags, the URL is fuzzily checked and tags such as _theme_, _plugin_, _config_ and _omf_ are added. The tag ~~_orphan_~~ **custom** is added by default as well. Finally, the author is generated by retrieving the e-mail or username of the author of the first commit in the plugin's repository. Closes #9 and #14.

* ~~Change `--path-in-cache` to `--translate.` This function translates an name or supported URL/URL variation into a path inside `$fisher_cache`. This allows you to treat plugins installed via custom URLs almost like regular plugins if they are installed. Closes #8.~~

* Fix a bug where `mktemp` would fail in some systems. Closes #7. Thanks @tobywf.

* Add [CODE_OF_CONDUCT](CODE_OF_CONDUCT.md). Closes #6.

* Fisherman can now unload themes within the same shell, without having to restart the session. Closes #5.

* Fisherman can now load themes within the same shell, without having to restart the session using `exec fish`. Shoddy themes, for example those failing to declare global variables with the `-g` flag still require the session to be reset. See [**related**](https://github.com/oh-my-fish/theme-bobthefish/pull/19). Closes #4.

* Move `getopts` implementation to `share/getopts.awk`. Closes #3.

* Support dots inside URIs in `fisher --validate`. Closes #2.

* Refactor and improve tests for `install`, `update` and `uninstall`.


## [0.1.0][v010] - 2016-01-01

* Initial commit.


:anchor:

<!--  Links -->

[v040]: https://

[v031]: https://github.com/fisherman/fisherman/commit/a0fe0b339df2fe70a0ba1a5e28dcd7449582742b

[v030]: https://github.com/fisherman/fisherman/commit/19758f98ab349e81d73fa8d813d06ee00a0fcb24

[v020]: https://github.com/fisherman/fisherman/commit/54212e1cbce66c7671baa045653efe912dbb4b77

[v010]: https://github.com/fisherman/fisherman/commit/3386ed052ae4a84338c340d37b98c1742f8a45f6
