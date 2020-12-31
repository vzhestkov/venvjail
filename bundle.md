## Python & Salt bundle

### 1. Created first builds of python38 for Ubuntu 20.04/18.04/16.04
There are a number of issues fixed and some issues still present, but in general build is successful for all distributions.

Fixed issues:
- `dash` is used with the link to `sh` instead on `bash`. It leads to the improper macro resolving by `debbuild` (text substitution with `%(...)` often produces empty string). Fixed by creating `dash2bash` dummy package (it's just fixing `sh` link to `bash` on installation, need to be added as a required for `deb` package build).
- 
