matrix-archiver
===============

If you want to add a room to be archived, please ensure it is public and the history is available to "Anyone", then open an issue with the internal ID (starts with !).

What it does
------------

*   Scoops the **entire** history of any *public, un-encrypted* Matrix room.
*   De-duplicates edits – only the **latest** version of a message is kept.
*   Renders:

    *   links (`http…` or `[label](http…)`)
    *   inline code `` `like this` ``
    *   fenced blocks

*   Colour-codes each user, shows basic threading, tags `[edited]`.
*   Emits

    ```
    archive/<slug>/index.html    ← pretty view
    archive/<slug>/room_log.txt  ← plain text
    index.html                   ← directory of rooms
    ```

Made for GitHub Actions + GitHub Pages but works anywhere there's python3.

---

Set-up (GitHub Pages)
---------------------

Create .env file from .env.example.

```sh
cp .env.example
```

Populate .env file with the following values:

| repo secret | what to put in it                                                |
|-------------|------------------------------------------------------------------|
| `MATRIX_HS` | homeserver URL, e.g. `https://matrix.example.org`                |
| `MATRIX_USER` | full bot ID, e.g. `@archiver:example.org`                      |
| `MATRIX_TOKEN` | long-lived access-token for that user obtained from Element > All Settings > Help & About > Advanced > Access Token |
| `MATRIX_ROOMS` | **space-separated internal room-IDs**, e.g.<br>`!abc:example.org !def:example.org` |

Commit the supplied workflow from `.github/workflows/` and you’re done:

* nightly cron → pulls fresh history  
* commits the new artefacts  
* deploys to `gh-pages`

---

Local run (one-off) macOS
-------------------

* Ensure the following before installing matrix-commander: https://github.com/8go/matrix-commander/blob/master/PyPi-Instructions.md

```bash
brew install cmake libolm dbus libmagic

# https://github.com/pyenv/pyenv?tab=readme-ov-file#homebrew-in-macos
brew update
brew install pyenv
alias brew='env PATH="${PATH//$(pyenv root)\/shims:/}" brew'


echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init - zsh)"' >> ~/.zshrc

exec "$SHELL"

xcode-select --install
brew install openssl readline sqlite3 xz zlib tcl-tk@8 libb2

pyenv install -l
# use 3.11.12 instead of 3.13.3 otherwise it generates error when installing dependencies
pyenv install 3.11.12
pyenv global 3.11.12
source ~/.zshrc

pip3 install poetry
CMAKE_POLICY_VERSION_MINIMUM=3.31 pip3 install matrix-commander

# fetch all messages instead of just the last 10,000 `LISTEN_MODE=all`
# fetch limited messages in tail mode `TAIL_N=5000`
TAIL_N=100 python3 scripts/update.py
# #graypaper:polkadot.io - archive
open ./archive/_21ddsEwXlCWnreEGuqXZ_3Apolkadot.io/index.html
# #jam:polkadot.io - archive
open ./archive/_21wBOJlzaOULZOALhaRh_3Apolkadot.io/index.html
# JAM Implementers room - archive
# FIXME - unable to archive since that room is encrypted _21KKOmuUpvYKPcniwOzw_3Amatrix.org
