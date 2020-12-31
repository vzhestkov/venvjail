## Python & Salt bundle

### 1. Created first builds of python38 for Ubuntu 20.04/18.04/16.04
There are a number of issues fixed and some issues still present, but in general build is successful for all distributions.

Fixed issues:
- A list of `Substitute:` defined in the project configuration (systemsmanagement:saltstack:bundle:python) to solve Ubuntu dependencies.
- `dash` is used with the link to `sh` instead on `bash`. It leads to the improper macro resolving by `debbuild` (text substitution with `%(...)` often produces empty string). Fixed by creating `dash2bash` dummy package (it's just fixing `sh` link to `bash` on installation, need to be added as a required for `deb` package build).
- `bcond` macroses not working the right way by default. Most probably due to the missing `%global` difinitions of appropriate values.
  Was solved by workaround in `spec`:
  ```
  %if "%_repository" == "xUbuntu_20.04" ||  "%_repository" == "xUbuntu_18.04" || "%_repository" == "xUbuntu_16.04"
  %global with_base 1
  %global with_general 1
  %global with_profileopt 1
  ...
  %endif
  ```
  It's better to move it to the project configuration, but I didn't find the way to define it there.
- The packages containing rpm macro was fixed with `_rpmmacrodir` definition in project configuration to move macro files to `/usr/lib/debbuild/macros.d` (for `debbuild`) and define it to `/usr/lib/rpm/macros.d` (for `rpmbuild`). `python-rpm-macros` and `fdupes` should work fine this way. (The previous definition of `%fdupes` in project configuration is not functional with `debbuild`).
- Dec 29 all the builds of `deb` packages failed due to updated `debbuild`. Find out the root cause: macro was not resolved properly in general and stops resolving `%_topdir` at the beggining of package building. Asked Neal Gumpta to revert this change as all of `deb` build packages were affected (https://build.opensuse.org/package/revisions/Debian:debbuild/debbuild). Revision 38 was broken due to incorrect change in the commit https://github.com/debbuild/debbuild/commit/cf2f32bfc688afab0b91c6d68d5fa87bf876e6cd
- `python38` package build was failed for all of the repos at Dec 29 also, but didn't find the root cause as it returned to normal at Dec 30, the sympthoms was missing dependencies for `doc` multibuild target which was actually disabled long time before. The only possible reason is impoper `bcond` macro resolution for `rpmbuild`, but don't know what caused it. Now it's fine.
- Some extra difinitions required by `python38` to make it working with `debbuild` (Now in `python38` `spec`, but it's better to move to project configuration):
  ```
  %define _gnu -gnu
  %define _lib lib64
  %define _defaultlicensedir %{_datadir}/licenses
  %define _libexecdir %{_exec_prefix}/lib
  ```

Existing issues:
 - Most probably multibuild is not working the same way for `debbuild` as for `rpmbuild`. It's related to 2nd element from the previous list, but not directly. I suppose the root cause is the different way of processing `%files` for the named packages. `python38` and `python38:base` build resulting the same list of `deb` packages, but different sizes. Some of the `deb` packages shouldn't be build for `python38:base`, but for `python38` and visa versa.
 - `debbuild` is not processing `%exclude` in `%files`, it could be the root cause of the previous issue.

In brief, it seems `debbuild` wors fine for simple `spec`, but not processing the complex `spec` with complex macroses.
